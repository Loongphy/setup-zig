# setup-zig

Fork of [mlugg/setup-zig](https://codeberg.org/mlugg/setup-zig) that runs the action on the **node24** Actions
runtime. Upstream still declares `node20`, which reached end-of-life in April 2026 and was removed from
GitHub-hosted runners on September 23, 2026 — see [actions/runner](https://github.com/actions/runner) and
upstream PR [codeberg.org/mlugg/setup-zig/pulls/55](https://codeberg.org/mlugg/setup-zig/pulls/55).

## Which ref to use

| Ref | Resolves to | Use when |
|---|---|---|
| `@node24-lts-runtime` | HEAD of the node24 patch branch | **recommended** — tracks the node24 runtime line, same content as `main` |
| `@main` | HEAD of `main` | always latest |
| `@v3` / `@v3.0.0` | release tags | immutable pin |
| `@<full commit SHA>` | exact commit | maximum reproducibility |

A branch ref like `@node24-lts-runtime` always resolves to the latest commit on that branch — updates are
picked up automatically on the next run. The fork tracks upstream: `main` carries the node24 patch on top of
the latest upstream commit.

## Deprecation

- The `v1.x`/`v2.x` tags and the floating `v1`/`v2` tags are **deprecated**: they declare the removed
  `node20` runtime and are kept only for history. Migrate to `@node24-lts-runtime` or `@v3`.
- There is no `node26` line yet — the Actions runner does not ship a `node26` runtime. When it does, a
  `node26-lts-runtime` branch and a new major tag will be added the same way.

## Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    name: Build and Test
    steps:
      - uses: actions/checkout@v5
      - uses: Loongphy/setup-zig@node24-lts-runtime
      - run: zig build test
```

This will automatically download Zig and install it to `PATH`.

You can use `version` to set a Zig version to download. This may be a release (`0.13.0`), a specific nightly
build (`0.14.0-dev.2+0884a4341`), the string `master` for the latest nightly build, or the string `latest`
for the latest full release. It can also refer to a [Mach nominated version][mach-nominated], such as
`2024.5.0-mach`. Finally, leaving the value empty (the default) will cause the action to attempt to resolve
the Zig version from the `minimum_zig_version` field in `build.zig.zon`, falling back to `latest` if that
isn't possible.

```yaml
  - uses: Loongphy/setup-zig@node24-lts-runtime
    with:
      version: 0.17.0
```

> [!WARNING]
> Mirrors, including the official Zig website, may purge old nightly builds at their leisure. This means
> that if you target an out-of-date nightly build, such as a `0.11.0-dev` build, the download may fail.

If you want to use one specific mirror, you can set it using the `mirror` option:

```yaml
  - uses: Loongphy/setup-zig@node24-lts-runtime
    with:
      mirror: 'https://pkg.machengine.org/zig'
```

Please don't do this unnecessarily; it's not nice to hammer one mirror. This mirror is not permitted to
be https://ziglang.org/download to avoid the official website being hit with large amounts of requests.
If you've experienced issues with a default mirror, please [open an issue][report-bad-mirror] on the Zig
website repository, which is where the list of mirrors is maintained.

If necessary, the caching of the global and local Zig cache directories can be disabled by setting the option
`use-cache: false`. Don't do this without reason: preserving the Zig cache will typically speed things up
and decrease the load on your Actions runners.

If you are using a [matrix strategy][matrix] for your workflow, you may need to populate the `cache-key` option
with all of your matrix variables to ensure that every job is correctly cached. Unfortunately, there is no way
for the Action to automatically distinguish jobs in a matrix. However, variables which select the runner platform
may be omitted from the `cache-key`, since the runner platform is included in the cache key by default.

Zig cache directories can get incredibly large over time. By default, this Action will clear the cache directory
once its size exceeds 2 GiB. This threshold can be changed by setting the `cache-size-limit` option to a different
value (in MiB); for instance, `cache-size-limit: 4096` for a 4 GiB limit. The limit can be disabled entirely by
setting `cache-size-limit: 0`.

[mach-nominated]: https://machengine.org/about/nominated-zig/
[matrix]: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/running-variations-of-jobs-in-a-workflow
[report-bad-mirror]: https://github.com/ziglang/www.ziglang.org/issues/new

## Details

This action attempts to download the requested Zig tarball from a set of mirrors, in a random order. As
a last resort, the official Zig website is used. The tarball's minisign signature is also downloaded and
verified to ensure binaries have not been tampered with. The tarball is cached between runs and workflows.

The global Zig cache directory (`~/.cache/zig` on Linux) is automatically cached between runs, and all
local caches are redirected to the global cache directory to make optimal use of this cross-run caching.

## Adding a mirror

The list of tarball mirrors is not in this repository; rather, the [community mirror list][mirrors] from
ziglang.org is used. If you are interested in hosting a mirror of your own, check out the
[documentation][host-mirror] on the Zig website repository. That way, your mirror can benefit not just
setup-zig, but also any other tooling which wants to fetch Zig!

[mirrors]: https://ziglang.org/download/community-mirrors/
[host-mirror]: https://github.com/ziglang/www.ziglang.org/blob/main/MIRRORS.md
