# Terraflow Studio — prebuilt images

Download ready-to-run container images from [Releases](https://github.com/ravirdoracle/terraflowstudio-images/releases)
or pull them from GitHub Container Registry.
This repository distributes existing images; it does not build the application.
Application source: [ravirdoracle/terraflowstudio](https://github.com/ravirdoracle/terraflowstudio).

## Container registry

Image: **`ghcr.io/ravirdoracle/terraflowstudio-images:latest`**

The registry image is the same **Linux ARM64** prebuilt image as the released
archive. It is loaded, checked, tagged, and pushed without rebuilding the app.
The release tag `v2026.11.07` is also available to pin this version.

```sh
# If the package is private, authenticate with an authorized GitHub account
# and a personal access token (classic) with read:packages at the password prompt.
podman login ghcr.io --username YOUR_GITHUB_USERNAME
podman pull ghcr.io/ravirdoracle/terraflowstudio-images:latest
podman tag ghcr.io/ravirdoracle/terraflowstudio-images:latest localhost/odb-terraform-app:latest
```

Public packages do not require login. After pulling and tagging, skip to step 3
below to set your login credentials and run the app. Docker users can substitute
`docker` for `podman`.

The **Publish prebuilt image to GHCR** workflow runs when a release is published,
or manually through **Actions → Run workflow** with a release tag. It verifies the
archive checksum and confirms the pushed image ID matches the loaded archive.

## Available download

| Release | File | Platform |
| --- | --- | --- |
| [v2026.11.07](https://github.com/ravirdoracle/terraflowstudio-images/releases/tag/v2026.11.07) | `odb-terraform-app-arm64.tar` | Linux ARM64 |

The ARM64 image runs on Apple Silicon through Podman/Docker Desktop and on Linux
ARM64 hosts. This download does not include an AMD64 image for Intel/AMD hosts.
The archive was already built and tested; no Dockerfile, source checkout, or
build command is needed to run it.

## 1. Download and verify

Download both files from the release:

- `odb-terraform-app-arm64.tar`
- `odb-terraform-app-arm64.tar.sha256`

Save them in the same directory. On macOS, verify the checksum:

```sh
shasum -a 256 -c odb-terraform-app-arm64.tar.sha256
```

On Linux, use `sha256sum -c odb-terraform-app-arm64.tar.sha256`.

## 2. Load the image

Start the Podman machine in **Podman Desktop → Settings → Resources** if it is
stopped, then run:

```sh
podman load -i odb-terraform-app-arm64.tar
podman images --filter reference=localhost/odb-terraform-app:latest
```

In Podman Desktop, use **Images → Load**, not **Import Containers**.
The loaded image name is `localhost/odb-terraform-app:latest`.

Docker users can replace `podman` with `docker` in the load, run, logs, and
container-management commands below.

## 3. Configure the login

Download [.env.example](https://github.com/ravirdoracle/terraflowstudio-images/blob/main/.env.example)
from this repository, save it locally, then copy it to `.env`:

```sh
cp .env.example .env
```

Set `APP_AUTH_USER` and a strong `APP_AUTH_PASSWORD` in `.env`. For example,
`openssl rand -hex 24` generates a password you can paste into the file.
Use plain values without surrounding quotes for Podman/Docker env files.

Both login values must be set together; incomplete credentials block requests.
Your browser prompts for this login when you open the app. The image contains no
account passwords or API keys. Keep your `.env` private.

LLM integration is optional. Leave the API key blank to use Terraform generation
without AI features.

## 4. Run on localhost or the host IP

From the directory containing your `.env`:

```sh
podman run -d \
  --name terraflowstudio \
  --restart unless-stopped \
  -p 0.0.0.0:8080:5000 \
  --env-file .env \
  -v terraflowstudio-data:/app/data \
  localhost/odb-terraform-app:latest
```

Open:

- On the container host: **http://localhost:8080**
- From another device on the same reachable network: **http://HOST_IP:8080**

Replace `HOST_IP` with the LAN IP of the computer running Podman/Docker, not the
Podman VM's IP. On a Mac using Wi-Fi, `ipconfig getifaddr en0` usually shows it.
The host firewall and network policy must allow inbound TCP 8080. If localhost
works but IP access fails, check those host settings; publishing a container port
does not override them.

Basic authentication protects the app's pages and APIs. Plain HTTP is suitable
only for a trusted private network; use an HTTPS reverse proxy for untrusted
networks so login credentials are encrypted in transit.

For access only from the container host, use `-p 127.0.0.1:8080:5000` instead.

## Manage the container

```sh
podman ps --filter name=terraflowstudio
podman logs --tail 50 terraflowstudio
podman stop terraflowstudio
podman start terraflowstudio
```

The `terraflowstudio-data` volume keeps saved customer configurations when a
container is recreated. Reuse that volume when upgrading. Update the host `.env`
and recreate the container to change its environment settings.

## Release details

- Image configuration ID: `3544bf6ff785253df7ea1ea431dedd598ae47bb1a900f360fd253962f2ebcac8`
- Platform: `linux/arm64`
- Image tag after loading: `localhost/odb-terraform-app:latest`
- Archive size: 1,370,888,704 bytes (about 1.28 GiB)
- Validation: Gunicorn startup and authenticated HTTP 200; anonymous settings
  requests receive HTTP 401.

Archives are attached to GitHub Releases, rather than committed to Git history.
