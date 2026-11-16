# Terraflow Studio — v2026.11.16

Linux **x86_64 / AMD64** and **ARM64** container images. The AMD64 image is built
on a native x86_64 runner from the released ARM64 application files, with source
parity and runtime feature checks. The ARM64 image matches the image running locally.

## Linux x86_64 / AMD64 support

- Added `odb-terraform-app-amd64.tar` and its SHA-256 checksum for Intel/AMD Linux hosts.
- The registry tags `v2026.11.16` and `latest` support both `linux/amd64` and `linux/arm64` and select the appropriate architecture automatically.
- To explicitly select x86_64: `podman pull --platform linux/amd64 ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16`.
- Both architectures include the same Active Data Guard Terraform/CLI features and UI changes below.

## New feature: OCI Active Data Guard

Open **OCI DG → C — Active Data Guard → Add Data Guard Association**.

- **Create standby and associate:** generate configuration to create a standby database from an existing primary on an existing Exadata VM cluster or supported bare metal/Exadata DB system. Active Data Guard is enabled by default and can be toggled on the card.
- **Check existing database pair:** generate a read-only lookup that reports association details, roles, state, redo apply lag/rate and whether Active Data Guard is enabled. This does not create an association between two independently created databases or enable Active Data Guard on an existing pair. Data Guard Group configurations use a separate API.
- **Generate Terraform/OpenTofu, OCI CLI, or both:** choose the output on each association card. Preview the files in the app and download them in the ZIP. CLI output includes `cli/run.sh`, `cli/request.json` and run instructions; run it with `bash run.sh` after configuring OCI CLI authentication.
- **Configure primary and standby details:** select the primary OCI region, enter the primary database OCID and target VM cluster/DB system OCID, and optionally specify the standby DB Home, unique name and SID prefix.
- **Keep passwords out of generated files:** Terraform and CLI creation prompt for the existing primary database admin password at runtime. The app does not collect that password.
- **Create with supported defaults:** creation uses `MAXIMUM_PERFORMANCE` protection and `ASYNC` transport. Apply and verify networking first, and check service compatibility, capacity, permissions and Active Data Guard licensing.
- **Terraform deletion protection:** generated creation configuration includes `prevent_destroy`, because provider deletion terminates the standby DB Home/DB system. Read the generated guide before applying. CLI creation has no Terraform state or lifecycle protection; choose one creation workflow per standby.

## UI changes

- Removed the AI prompt template button, library, popup and styles.
- Removed the AI prompt input/Ask bar from AWS, GCP, Azure, OCI Data Guard and OCI DB pages.
- Moved spreadsheet import and template download below the customer row on AWS, GCP and Azure to match OCI Data Guard.

## Download and run

Download the archive for your host and its `.sha256` file:

| CPU | Archive |
| --- | --- |
| Intel/AMD Linux x86_64 | `odb-terraform-app-amd64.tar` |
| Apple Silicon / Linux ARM64 | `odb-terraform-app-arm64.tar` |

For Intel/AMD Linux:

```sh
sha256sum -c odb-terraform-app-amd64.tar.sha256
podman load -i odb-terraform-app-amd64.tar
podman run -d --name terraflowstudio --restart unless-stopped \
  -p 0.0.0.0:8080:5000 --env-file .env \
  -v terraflowstudio-data:/app/data localhost/odb-terraform-app:latest
```

For ARM64, substitute `arm64` in the filenames; on macOS use `shasum -a 256 -c`.

Set both `APP_AUTH_USER` and a strong `APP_AUTH_PASSWORD` in your private `.env` first.
Open `http://localhost:8080` or `http://HOST_IP:8080` on a reachable private network.
Docker users can substitute `docker` for `podman`. In Podman Desktop, choose **Load**.
Reuse your existing data volume when upgrading; do not delete it. Stop and rename
an existing container before reusing its name. Refresh the browser with Cmd+Shift+R
(macOS) or Ctrl+Shift+R after upgrading.

## Container registry

The ARM64 archive is loaded without rebuilding. The AMD64 counterpart is built natively
from the released application files. Both are published under these platform-aware tags:

- `ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16`
- `ghcr.io/ravirdoracle/terraflowstudio-images:latest`

```sh
podman pull ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16
```

## Validation and platform

- Platforms: `linux/amd64` (Intel/AMD x86_64 Linux) and `linux/arm64` (Apple Silicon through Podman/Docker Desktop, or Linux ARM64).
- ARM64 image configuration ID: `faddbd0208747eba6696cf08e8a9ccae1b406163defb2491af47ff640ad994c0`.
- ARM64 archive size: 1,373,583,872 bytes.
- AMD64 image configuration ID: `sha256:e22de7bbcb103e651260faaa443cb3b23cd24800414ae257e59a2ccef0d6ef6b`.
- AMD64 archive size: 317,847,040 bytes.
- AMD64 archive SHA-256: `9a96ad28b6cfa117ba63e8fc6043cad812119575c0b2bb8671036e528a9d4a31`.
- Native x86_64 build verified application source-file parity, authentication, all five product pages, toolbar order, Data Guard Terraform/CLI generation, and both bundled Terraform/OpenTofu binaries.
- All three generated Terraform paths validated against OCI provider 9.11.0.
- 121 targeted Data Guard/CLI/architecture tests passed; page rendering, JavaScript syntax and inline handlers checked after UI removals.
- Exact release image checked offline: unauthenticated pages/settings return 401; authenticated product pages return 200; no bundled `.env`, `.aws`, `.codex` or `.agents`; toolbar order and CLI generation verified.
- No OCI infrastructure was created during validation.

## Notes

The runtime image includes Terraform and OpenTofu. OCI provider plugins are downloaded
by `terraform init`; network access is required. The generated CLI scripts require
Bash, Python 3 and a configured OCI CLI on the machine where they are run.
Independently created databases cannot be associated by the existing-pair lookup.
Avoid running both Terraform and CLI creation for the same standby. CLI creation is
not tracked by Terraform state. One pre-existing full-suite test still expects eight
saved X11M configurations where the local data contains ten; this release does not
change that test or those saved configurations.
