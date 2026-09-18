# Changelog

## 1.0.0 (2026-09-18)


### Features

* add CI validation, release-please versioning, and renovate config ([0b5cd97](https://github.com/st0o0/github-workflows/commit/0b5cd9793f0c21a5de9c7303d81e10c4bbfbc943))
* add coverage report and optional locked-mode to dotnet-ci ([cb69b73](https://github.com/st0o0/github-workflows/commit/cb69b7355609f4cfce48ffcd28f236f944e681e7))
* add dotnet docker release, nuget release, and dotnet dev-build workflows ([73cf4c4](https://github.com/st0o0/github-workflows/commit/73cf4c44fd6192ed1aac8043376cd76cd3abadd8))
* add filesystem scan mode to security workflow ([d6e0d7f](https://github.com/st0o0/github-workflows/commit/d6e0d7f59374833e5c3a657a9e692b603349dfaf))
* add release-please manifest config ([70ef92f](https://github.com/st0o0/github-workflows/commit/70ef92f4d21c5527539da366690c473e6d321a94))
* add reusable workflows for Go, .NET, Docker, and docs ([38a6395](https://github.com/st0o0/github-workflows/commit/38a6395dfe94bc20028a4c4e71bb1aeb11fc65b4))
* attach nupkg to GitHub release, update readme examples ([4124bd0](https://github.com/st0o0/github-workflows/commit/4124bd0f79d92c882eab0205a65a769f8637af2a))
* cosign signing for dev builds, SHA-pin installer, add verify step ([31fe04b](https://github.com/st0o0/github-workflows/commit/31fe04bcd29d5b59ea194f0fc28fffbe0994b236))
* decouple release-please from build workflows ([0204520](https://github.com/st0o0/github-workflows/commit/0204520bbd870c02b91e0365eead5ca0779cb9b3))
* migrate docker workflows to multi-stage builds ([334a742](https://github.com/st0o0/github-workflows/commit/334a7429c66969e65571f4df248a86d406354b7a))
* modular build and docker workflows with pnpm support ([7f9f45d](https://github.com/st0o0/github-workflows/commit/7f9f45d36f840709e1db0e593c2883192af681ff))
* remove e2e job and input from go-ci workflow ([7985d03](https://github.com/st0o0/github-workflows/commit/7985d030c244ac9ac7c0df51b6960233446a7993))


### Bug Fixes

* add category to CodeQL analyze step ([4c09a72](https://github.com/st0o0/github-workflows/commit/4c09a72847dc2b9c2560ca37df6787ccd5be67bb))
* add explicit SARIF upload categories to prevent code scanning config mismatch ([43eb0d7](https://github.com/st0o0/github-workflows/commit/43eb0d751d4cf376678183dd4464fb09060dd8d6))
* add missing dotnet restore step before publish ([5954f08](https://github.com/st0o0/github-workflows/commit/5954f089cf9bc1955605984751f659304f7ee405))
* dotnet format takes solution as positional arg, not --solution ([4688245](https://github.com/st0o0/github-workflows/commit/4688245e149f93f3c195d8f308ab41bfd6c14653))
* pass runtime identifier to dotnet restore ([6a97173](https://github.com/st0o0/github-workflows/commit/6a97173f70b5309e91aee49ec6000499e2b14032))
* remove unnecessary pull-requests permission from nuget build job ([dcafd4e](https://github.com/st0o0/github-workflows/commit/dcafd4ee128ae61a46b4bf7899d02804f0b20143))
* review findings -- names, cache keys, injection, coverage, action versions ([f428a3e](https://github.com/st0o0/github-workflows/commit/f428a3e75f7ebb9f6fa63941e4f63f2529803260))
* separate restore/build/test steps, avoid redundant restores ([9f07035](https://github.com/st0o0/github-workflows/commit/9f07035f0bb542b56375e82613cdf1bf2a55351e))
* skip docs in trivy scan, harden security/dotnet-ci, drop dead workflows ([a45eeb7](https://github.com/st0o0/github-workflows/commit/a45eeb76f2050b117836b5a8fb225893d115dd89))
* skip SARIF upload when trivy.sarif does not exist ([7bbd8cd](https://github.com/st0o0/github-workflows/commit/7bbd8cd8e5334a278de893e4d79cc007e06a41f8))
* upgrade pnpm-ci to Node 24 and resolve pnpm version detection ([75b62fe](https://github.com/st0o0/github-workflows/commit/75b62fe7be1bb65903a6618e0d24b7fb587a211c))
* use --solution flag for dotnet test (MTP requires it) ([4dde2a2](https://github.com/st0o0/github-workflows/commit/4dde2a2e9b7dec4afc3fc30f260ad697d392b4b8))
* use --solution flag for dotnet test and dotnet format ([7a90dd6](https://github.com/st0o0/github-workflows/commit/7a90dd6377d73cd652e8a8aa42f12d2531341198))
* use positional args for dotnet restore/build/test (no --solution flag) ([924ff3a](https://github.com/st0o0/github-workflows/commit/924ff3add42943fd90faf74dcf7499d7b958681e))
* use secret instead of input for NUGET_USER ([d392ce8](https://github.com/st0o0/github-workflows/commit/d392ce87bbdc1fbe56bb3b3ecd934d3f0f95282e))


### Refactors

* merge security-docker and security-dotnet into single security workflow ([d9688cc](https://github.com/st0o0/github-workflows/commit/d9688cc308ef5f889bb8312e9da42c155a96d5ff))
* migrate docs workflow to pnpm ([2d65254](https://github.com/st0o0/github-workflows/commit/2d65254ec3c24276036c2c556893bae7f7a20380))
* remove nuget-audit from security workflow ([245dd2e](https://github.com/st0o0/github-workflows/commit/245dd2e8391a36d5f600a0e18f3872c581d006d2))
* rename jobs for cleaner GitHub check names ([c9c1ff6](https://github.com/st0o0/github-workflows/commit/c9c1ff620983db3676e939ea693231def4934191))
* rename pnpm-ci to frontend-ci ([39d5778](https://github.com/st0o0/github-workflows/commit/39d577851236c34934ba5bb9556dbba9f911c50d))
* rename workflows to consistent naming convention ([872ad5e](https://github.com/st0o0/github-workflows/commit/872ad5ed20b71fcc1ed39371aa0de519b7692df9))
* switch dotnet-release-nuget from GitVersion to release-please ([e02fcd4](https://github.com/st0o0/github-workflows/commit/e02fcd4bf74968cb1dbb5fcab3c23ea1120ea313))
* switch nuget publish from API key to Trusted Publishing (OIDC) ([cef31be](https://github.com/st0o0/github-workflows/commit/cef31be8d2975eecb1cb8c7b130a86efc540b79d))
