# Prusa Connect Local for Unraid

Self-hosted local Prusa Connect server for managing Prusa 3D printers on your LAN without relying on Prusa's cloud. This Unraid Community Applications template uses the **official Prusa Research image**, `ghcr.io/prusa3d/prusa-connect`.

This repository maintains the Unraid template only, not the application or Docker image. It is not affiliated with Prusa Research.

## Installation

1. Install from Unraid Community Applications once approved, or import [templates/prusa-connect.xml](templates/prusa-connect.xml) for testing.
2. The default WebUI and printer API port is **8080/TCP**. If the host port is occupied by another application (for example, qBittorrent or SABnzbd), change the **Host Port** in the Unraid installer. Keep the container port at 8080.
3. Review the host paths in the installer before clicking Apply. The writable settings mount is `/config/config.d`; other mounts are `/data`, `/files`, `/storage`, and `/tmp`.
4. Create the writable paths and give UID/GID **10001:10001** ownership before starting (see below).
5. Start the container and open its **Logs**. Find the one-time administrator setup code shown as `setup_code=XXXX-XXXX-XXXX`.
6. Open the WebUI and enter the code to claim administrator ownership; complete the first-run wizard.
7. **Restart Prusa Connect after completing the wizard**, as recommended in Prusa's documentation.

### Permissions and persistent storage

The official image runs as UID/GID `10001:10001`. It does not use `PUID` or `PGID`. The container uses a read-only root filesystem, with writable bind mounts at `/config/config.d`, `/data`, `/files`, `/storage`, and `/tmp`. In a verified fresh Unraid install the host directories were created as `99:100` with mode `0755`, causing `mkdir config/config.d: permission denied` and a restart loop. The template cannot automatically correct host-directory ownership.

**Review the actual host paths in Unraid before applying these commands.** For example, if all mappings use `/mnt/user/appdata/prusa-connect`:

```bash
APPDATA=/mnt/user/appdata/prusa-connect
mkdir -p "$APPDATA"/{config.d,data,files,storage,tmp}
chown -R 10001:10001 "$APPDATA"/{config.d,data,files,storage,tmp}
```

If you customized the paths in the installer, substitute **each actual host path** in the commands above. Restrict `chown` to the Prusa Connect directories, never your shared appdata root or directories used by other containers. On a fresh install, prepare these directories **before** first startup; if you already saw a restart loop, stop the container, correct their ownership, and start it again.

An explicit `/config/config.yaml` is optional on first launch. The **Basic** setup wizard can create persistent settings and activate local upload storage using `/storage` without a base file. If you later supply a base `config.yaml`, bind-mount it separately as a read-only file at `/config/config.yaml` and ensure UID 10001 can read it. Restart after configuration changes.

**Keep the setup code private.** It authorizes the initial administrator claim. If the server restarts before setup is completed, it generates a new one.

## Container defaults

| Setting | Value |
|---|---|
| Repository | `ghcr.io/prusa3d/prusa-connect:0.1.0-beta.1` |
| Host port | `8080` (user-configurable) |
| Container port | `8080/tcp` |
| Network | Bridge |
| Runtime permissions | Non-root; read-only filesystem; drop all capabilities; no new privileges |
| Persistent paths | `/config/config.d`, `/data`, `/files`, `/storage`, `/tmp` |

**Image tag note:** Prusa does not publish a `latest` tag for prereleases, and the first available published image is currently `0.1.0-beta.1`. This temporary pin ensures fresh installations work. When Prusa publishes its first stable release, the template will switch to `ghcr.io/prusa3d/prusa-connect:latest`. After that, Unraid can detect newer official images and offer updates from the Docker tab. Users can manually select an exact prerelease version in the Repository field.

### Existing installations

Changing an existing container from a `/config` bind mount to `/config/config.d` **does not migrate files automatically**. Leave working installations unchanged unless you explicitly move the prior `config.d` contents and update the mount. The new template layout is intended for fresh installations.

## Support

- [Unraid template issues](https://github.com/bleugray/prusa-connect-unraid/issues)
- [Official Prusa Connect source](https://github.com/prusa3d/prusa-connect)
- [Official Docker image](https://github.com/prusa3d/prusa-connect/pkgs/container/prusa-connect)
- [Official container documentation](https://github.com/prusa3d/prusa-connect/blob/main/docs/src/content/docs/container.md)

The icon is loaded from Prusa's official repository and is not redistributed here.
