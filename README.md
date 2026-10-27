# decypher-releases

Release assets of [Decypher](https://github.com/Neuvem), a code-intelligence tool for JVM code. The source is not in this repository: it holds
the files that `decypher runtime install` downloads, and the workflow that signs their checksums.

## Releases

| Tag | What | Signed by |
|---|---|---|
| `v<version>` | The JVM runtime that `decypher runtime install` downloads. | `.github/workflows/sign.yml` |
| `decypher-v<version>` | The `decypher` command-line binaries for linux, macOS and windows. | `.github/workflows/sign-cli.yml` |

## The JVM runtime (`v<version>`)

What a runtime release contains:

| File | What it is |
|---|---|
| `decypher-jvm-<version>-<os>-<arch>.tar.zst` | The JVM runtime (a jlink image with the Decypher extractor daemon) for one platform. |
| `decypher-jvm-<version>.cdx.json` | CycloneDX SBOM of the daemon and its dependencies. |
| `decypher-jvm-<version>.sha256` | SHA-256 of every archive and of the SBOM. |
| `decypher-jvm-<version>.sha256.sigstore.json` | Sigstore bundle: a keyless signature of the `.sha256` file. |

The `decypher` binary pins the digest of the archive for its platform and refuses an archive that does not match, so a download from
anywhere is checked against a digest compiled into the program.

### Verifying the runtime

```sh
cosign verify-blob --bundle decypher-jvm-2.0.0.sha256.sigstore.json \
  --certificate-identity https://github.com/Neuvem/decypher-releases/.github/workflows/sign.yml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com decypher-jvm-2.0.0.sha256
sha256sum -c decypher-jvm-2.0.0.sha256        # shasum -a 256 -c on macOS
```

### What the runtime signature says

The archives are built by the maintainers on their own machines (jlink needs the target platform's JDK) and uploaded to a draft release.
`.github/workflows/sign.yml` then downloads them, checks each against the digest recorded when it was built, and signs the combined checksum
file with this workflow's identity. The signature therefore tells you that these exact bytes were the release's contents when this
workflow signed them. It does not claim that GitHub Actions built them.

## The `decypher` binaries (`decypher-v<version>`)

| File | What it is |
|---|---|
| `decypher-<os>-<arch>[.exe]` | The command-line tool: linux-amd64, linux-arm64, darwin-amd64, darwin-arm64, windows-amd64. A single static binary. |
| `decypher-<version>.cdx.json` | CycloneDX SBOM of the Go modules linked into the binaries. |
| `SHA256SUMS` | SHA-256 of every file above. |
| `SHA256SUMS.sigstore.json` | Sigstore bundle: a keyless signature of `SHA256SUMS`. |

These are built by GitHub Actions in the (private) source repository from the tagged commit, uploaded here as a draft, and signed by
`.github/workflows/sign-cli.yml` in this repository once it has checked the files against `SHA256SUMS`.

```sh
v=2.0.0
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity https://github.com/Neuvem/decypher-releases/.github/workflows/sign-cli.yml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com SHA256SUMS
sha256sum -c --ignore-missing SHA256SUMS     # shasum -a 256 -c on macOS
./decypher-linux-amd64 version
```
