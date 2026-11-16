# Terraflow Studio — container images

Download ready-to-run images from [Releases](https://github.com/ravirdoracle/terraflowstudio-images/releases)
or pull from GitHub Container Registry. Application source repository:
[ravirdoracle/terraflowstudio](https://github.com/ravirdoracle/terraflowstudio).

This repository distributes the ARM64 prebuilt image and builds its AMD64 counterpart
on a native Linux x86_64 runner using the application files extracted from that
released ARM64 image. Source-file hashes are checked for parity between architectures.

## Platforms and downloads

| Platform | Typical hosts | Release archive |
| --- | --- | --- |
| Linux x86_64 / AMD64 | Intel and AMD Linux servers and PCs | `odb-terraform-app-amd64.tar` |
| Linux ARM64 | Apple Silicon through Podman/Docker Desktop, Linux ARM64 hosts | `odb-terraform-app-arm64.tar` |

Each archive has a matching `.sha256` checksum file. The image inside either archive
is named `localhost/odb-terraform-app:latest`.

Latest release: **[v2026.11.16](https://github.com/ravirdoracle/terraflowstudio-images/releases/tag/v2026.11.16)**.
See [release notes](RELEASE_NOTES_v2026.11.16.md) for Active Data Guard Terraform/CLI
output and UI improvements. The earlier ARM64-only release
[v2026.11.07](https://github.com/ravirdoracle/terraflowstudio-images/releases/tag/v2026.11.07)
remains available.

## 1. Get the image

### Pull from the registry

```sh
podman pull ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16
podman tag ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16 localhost/odb-terraform-app:latest
```

The registry automatically selects Linux AMD64 or ARM64 for the host. The `latest`
tag is also available. To explicitly select x86_64:

```sh
podman pull --platform linux/amd64 ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16
```

Public images do not require login. Docker users can substitute `docker` for
`podman` throughout these instructions.

### Or download and load an archive

Download the archive for your CPU and its checksum from the release. Save them in
the same directory. For Intel/AMD Linux:

```sh
sha256sum -c odb-terraform-app-amd64.tar.sha256
podman load -i odb-terraform-app-amd64.tar
```

For Apple Silicon / ARM64 on macOS:

```sh
shasum -a 256 -c odb-terraform-app-arm64.tar.sha256
podman load -i odb-terraform-app-arm64.tar
```

On Linux ARM64, use `sha256sum` with the ARM64 checksum. Start the Podman machine
in Podman Desktop if needed. In Podman Desktop, choose **Images → Load**, rather
than **Import Containers**.

## 2. Configure the login

Download [.env.example](.env.example), save it locally and copy it to `.env`:

```sh
cp .env.example .env
```

Set both `APP_AUTH_USER` and a strong `APP_AUTH_PASSWORD`. You can generate a
password with `openssl rand -hex 24`. Use plain values without surrounding quotes
for Podman/Docker env files. The browser prompts for these credentials when you
open the app. Both values must be set together; incomplete credentials block requests.
The images contain no account passwords or API keys. Keep your `.env` private.
LLM integration is optional; Terraform and CLI generation work without an API key.

## 3. Run on localhost or the host IP

```sh
podman run -d \
  --name terraflowstudio \
  --restart unless-stopped \
  -p 0.0.0.0:8080:5000 \
  --env-file .env \
  -v terraflowstudio-data:/app/data \
  localhost/odb-terraform-app:latest
```

Open **http://localhost:8080** on the host or **http://HOST_IP:8080** from another
reachable device. Use the host computer's LAN IP, not the Podman VM's IP.
Host firewall and network policy must permit inbound TCP 8080; publishing the port
does not override them. For host-only access, use `-p 127.0.0.1:8080:5000`.
Use HTTPS through a reverse proxy when the app is exposed beyond a trusted private
network: HTTP Basic authentication credentials are not encrypted by plain HTTP.

The `terraflowstudio-data` volume preserves saved configurations. Reuse it when
upgrading. Stop and rename an existing container before reusing its name; do not
delete its data volume. Recreate the container to apply changes to `.env`.

```sh
podman logs --tail 50 terraflowstudio
podman stop terraflowstudio
podman start terraflowstudio
```

## Publishing workflows

- **Publish prebuilt image to GHCR** loads and checks an ARM64 archive from a release.
- **Build Linux AMD64 from released application** takes a release tag and an ARM64
  manifest digest. It extracts the matching app files, builds on native x86_64,
  verifies source parity and current features, uploads the AMD64 archive/checksum,
  and publishes combined AMD64/ARM64 registry tags.

Archives are release assets; they are not committed to Git history. Run the AMD64
workflow after publishing a new ARM64 release to offer both architectures.
