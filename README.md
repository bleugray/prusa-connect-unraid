# Prusa Connect Local for Unraid

Self-hosted local Prusa Connect server for managing Prusa 3D printers on your LAN without relying on Prusa's cloud. This Unraid Community Applications template uses the **official Prusa Research image**, `ghcr.io/prusa3d/prusa-connect`.

This repository maintains the Unraid template only, not the application or Docker image. It is not affiliated with Prusa Research.

## Installation

1. Install from Unraid Community Applications once approved, or import [templates/prusa-connect.xml](templates/prusa-connect.xml) for testing.
2. The default WebUI and printer API port is **8080/TCP**. If the host port is occupied by another application (for example, qBittorrent or SABnzbd), change the **Host Port** in the Unraid installer. Keep the container port at 8080.
3. Review all host paths before installing. Unraid may substitute its configured appdata location for the `/config` mount.
4. Create the writable paths and give UID/GID **10001:10001** ownership before starting (see below).
5. Start the container and open its **Logs**. Find the one-time administrator setup code shown as `setup_code=XXXX-XXXX-XXXX`.
6. Open the WebUI and enter the code to claim administrator ownership; complete the first-run wizard.
7. **Restart Prusa Connect after completing the wizard**, as recommended in Prusa's documentation.

### Permissions and persistent storage

The official image runs as UID/GID `10001:10001`. It does not use `PUID` or `PGID`. The container uses a read-only root filesystem, with writable mounts at `/config`, `/data`, `/files`, `/storage`, and `/tmp`.

**Review the actual host paths in Unraid before applying these commands.** For example, if all mappings use `/mnt/user/appdata/prusa-connect`:

```bash
APPDATA=/mnt/user/appdata/prusa-connect
mkdir -p "$APPDATA"/{config/config.d,data,files,storage,tmp}
chown -R 10001:10001 "$APPDATA"/{config,data,files,storage,tmp}
```

Unraid may replace the host mapping for `/config` with its own default appdata path, such as `/mnt/cache/appdata/Prusa-Connect`. In that case, ensure that **that directory and its `config.d` child** are writable and traversable by UID/GID 10001, rather than assuming `$APPDATA/config`.

An explicit `/config/config.yaml` is optional on first launch; if supplied, it must be readable by the container user. The server reads configuration on startup, so restart after changes.

**Keep the setup code private.** It authorizes the initial administrator claim. If the server restarts before setup is completed, it generates a new one.

## Container defaults

| Setting | Value |
|---|---|
| Repository | `ghcr.io/prusa3d/prusa-connect:0.1.0-beta.1` |
| Host port | `8080` (user-configurable) |
| Container port | `8080/tcp` |
| Network | Bridge |
| Runtime permissions | Non-root; read-only filesystem; drop all capabilities; no new privileges |
| Persistent paths | `/config`, `/data`, `/files`, `/storage`, `/tmp` |

The image is pinned to a beta while we validate the template. The maintainer can update the image reference as compatible official releases become available.

## Support

- [Unraid template issues](https://github.com/bleugray/prusa-connect-unraid/issues)
- [Official Prusa Connect source](https://github.com/prusa3d/prusa-connect)
- [Official Docker image](https://github.com/prusa3d/prusa-connect/pkgs/container/prusa-connect)
- [Official container documentation](https://github.com/prusa3d/prusa-connect/blob/main/docs/src/content/docs/container.md)

The icon is loaded from Prusa's official repository and is not redistributed here.
