# Terraflow Studio — v2026.11.16

Linux ARM64 prebuilt container image, exported from the same image running locally.

## Added

- OCI Active Data Guard association tab: create a new standby on an existing Exadata VM cluster or supported bare metal/Exadata DB system.
- Read-only lookup of an existing database pair, including association status, roles and Active Data Guard details. This does not associate two independently created databases; Data Guard Group configurations use a separate API.
- Per-association output choice: Terraform/OpenTofu, OCI CLI, or both. CLI downloads include `run.sh`, `request.json` and run instructions.
- Runtime password prompts rather than passwords in generated files. Terraform creation includes `prevent_destroy`; read the generated guide before applying.

## UI changes

- Removed the AI prompt template button, library, popup and styles.
- Removed the AI prompt input/Ask bar from AWS, GCP, Azure, OCI Data Guard and OCI DB pages.
- Moved spreadsheet import and template download below the customer row on AWS, GCP and Azure to match OCI Data Guard.

## Download and run

Download `odb-terraform-app-arm64.tar` and its `.sha256` file, then:

```sh
shasum -a 256 -c odb-terraform-app-arm64.tar.sha256
podman load -i odb-terraform-app-arm64.tar
podman run -d --name terraflowstudio --restart unless-stopped \
  -p 0.0.0.0:8080:5000 --env-file .env \
  -v terraflowstudio-data:/app/data localhost/odb-terraform-app:latest
```

Set both `APP_AUTH_USER` and a strong `APP_AUTH_PASSWORD` in your private `.env` first.
Open `http://localhost:8080` or `http://HOST_IP:8080` on a reachable private network.
Docker users can substitute `docker` for `podman`. In Podman Desktop, choose **Load**.
Reuse your existing data volume when upgrading; do not delete it. Stop and rename
an existing container before reusing its name. Refresh the browser with Cmd+Shift+R
(macOS) or Ctrl+Shift+R after upgrading.

## Container registry

The release-published workflow loads this archive and publishes these tags without rebuilding:

- `ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16`
- `ghcr.io/ravirdoracle/terraflowstudio-images:latest`

```sh
podman pull ghcr.io/ravirdoracle/terraflowstudio-images:v2026.11.16
```

## Validation and platform

- Platform: `linux/arm64` (Apple Silicon through Podman/Docker Desktop, or Linux ARM64). No AMD64 archive is included.
- Image configuration ID: `faddbd0208747eba6696cf08e8a9ccae1b406163defb2491af47ff640ad994c0`.
- Archive size: 1373583872 bytes.
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
