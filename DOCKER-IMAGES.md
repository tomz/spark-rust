# sparkrust playground container releases

This repository publishes **binary container images only**. Engine source code is maintained outside this release repository and is not copied here.

## Editions

| Edition | GHCR tag | Kernels | Recommended memory |
|---|---|---|---:|
| Tiny | `ghcr.io/tomz/spark-rust-playground:tiny` | Python/PySpark, Spark SQL | 8 GiB |
| End-user | `ghcr.io/tomz/spark-rust-playground:enduser` | Python, Spark SQL, Scala, SparkR, TypeScript, Ruby | 12 GiB |
| Developer | `ghcr.io/tomz/spark-rust-playground:developer` | End-user kernels plus Rust/Evcxr and compiler toolchains | 16 GiB |

Every release tag is intended to be an OCI manifest containing native `linux/amd64` and `linux/arm64` images. Docker automatically chooses the host architecture.

## Local release artifacts

Validated images are exported into the gitignored `artifacts/` directory before publication. They are deliberately **not committed to Git** because GitHub source repositories are not a container registry and these archives are multi-gigabyte binaries.

Expected local layout:

```text
artifacts/
├── amd64/
│   ├── tiny.docker.tar.zst
│   ├── enduser.docker.tar.zst
│   └── developer.docker.tar.zst
├── arm64/
│   ├── tiny.docker.tar.zst
│   ├── enduser.docker.tar.zst
│   └── developer.docker.tar.zst
├── SHA256SUMS
└── images.tsv
```

## Pull and run after publication

```bash
docker pull ghcr.io/tomz/spark-rust-playground:tiny

docker run --rm --memory=8g --memory-swap=8g \
  -p 8010:8000 ghcr.io/tomz/spark-rust-playground:tiny
```

End-user and developer editions use ports 8009 and 8008 by convention:

```bash
docker run --rm --memory=12g --memory-swap=12g \
  -p 8009:8000 ghcr.io/tomz/spark-rust-playground:enduser

docker run --rm --memory=16g --memory-swap=16g \
  -p 8008:8000 ghcr.io/tomz/spark-rust-playground:developer
```

## Pre-publication gate

Do not push until all of the following are true for both architectures:

1. image architecture matches its immutable architecture tag;
2. JupyterHub login endpoint becomes ready;
3. every advertised kernelspec starts and executes a real query;
4. every bundled notebook executes all code cells in a fresh real kernel;
5. TPC-H 22/22 and TPC-DS 99/99 notebook cells pass;
6. no OneLake, Unity, HMS, Azure, Cloudflare, GitHub, SSH, or cluster credential is present in any image layer;
7. vulnerability scan, SBOM, provenance, and image digest are recorded;
8. a clean AMD64 host and clean ARM64 host can import and run the archived image;
9. GHCR credentials have `write:packages` and `read:packages` scopes;
10. publication is explicitly approved.

## Publication model

Images are published to GitHub Container Registry, not stored in the source repository:

```text
ghcr.io/tomz/spark-rust-playground:<flavor>-<date>-<git>-amd64
ghcr.io/tomz/spark-rust-playground:<flavor>-<date>-<git>-arm64
ghcr.io/tomz/spark-rust-playground:<flavor>-vX.Y.Z
```

The two architecture tags are immutable. The version tag is assembled with `docker buildx imagetools create`. Moving tags (`tiny`, `enduser`, `developer`) are updated only after clean-pull validation of the versioned manifest.
