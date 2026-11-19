# Prusa Connect Local for Unraid

Self-hosted Prusa Connect Local for managing Prusa 3D printers on your LAN without depending on Prusa's cloud. This Community Apps template installs the **official Prusa Research Docker image**, `ghcr.io/prusa3d/prusa-connect`.

This is an independently maintained **Unraid installation template**, not an official Prusa Research project.

> **IMPORTANT: Before clicking Apply in Unraid, prepare the host directories and correct their ownership.** Skipping this step causes Prusa Connect to repeatedly restart with `mkdir config/config.d: permission denied`. This behavior was reproduced in a clean Unraid 7.4.0 installation and resolved by changing ownership.

## Installation: step by step

### 1. Open the Unraid application installer

Install **Prusa-Connect** from Community Apps, or load [the Docker template](templates/prusa-connect.xml) directly.

The template defaults to host port **8080/TCP** and container port **8080/TCP**. If another application already uses host port 8080 (such as qBittorrent or SABnzbd), change **Host Port** to an available port, such as `8095`. **Do not change the container port**, which must remain 8080.

**Do not click Apply yet.**

### 2. Review all five host directory paths

Note the **host** paths shown for each of these mounts. The default installation uses:

| Unraid host directory | Container path | Purpose |
|---|---|---|
| `/mnt/user/appdata/prusa-connect/config.d` | `/config/config.d` | Login and authorization database, setup state, dynamic settings, history |
| `/mnt/user/appdata/prusa-connect/data` | `/data` | Application/extension data |
| `/mnt/user/appdata/prusa-connect/files` | `/files` | Application files |
| `/mnt/user/appdata/prusa-connect/storage` | `/storage` | Uploaded print files |
| `/mnt/user/appdata/prusa-connect/tmp` | `/tmp` | Temporary/scratch space |

If your Unraid system stores appdata under a cache pool or you edit the paths in the installer, **use the paths actually displayed on your server**. The commands in the next step assume you left all five paths at their defaults.

### 3. Create the directories and fix ownership BEFORE clicking Apply

Open **Unraid → Terminal** or connect with SSH. For the **default paths above**, run:

```bash
APPDATA="/mnt/user/appdata/prusa-connect"

mkdir -p "$APPDATA"/{config.d,data,files,storage,tmp}
chown -R 10001:10001 "$APPDATA"/{config.d,data,files,storage,tmp}

ls -ldn "$APPDATA"/{config.d,data,files,storage,tmp}
```

Each of those five directories should show owner UID `10001` and group GID `10001`. Do **not** use `chmod 777`. Do **not** run a recursive `chown` on your entire `/mnt/user/appdata` share or on directories belonging to other containers.

**If you changed any host path:** create and chown that exact directory instead of blindly running the default example. The container cannot repair its own host-side directory ownership.

**Why?** Unraid commonly creates appdata directories as `99:100` with permissions `0755`. Prusa's official image runs as non-root UID/GID `10001:10001`, does not support `PUID` or `PGID`, and requires writable directories to store its database. A Docker bind mount retains the host directory's ownership.

### 4. Start the container and obtain the one-time setup code

Return to Unraid's Docker installation form and click **Apply**.

After the container starts, open **Docker → Prusa-Connect → Logs** and locate a line containing:

```text
setup_code=XXXX-XXXX-XXXX
```

This is a **one-time administrator-claim code**. Enter it into the first-run browser wizard at your Unraid host IP and selected **Host Port** (for example, `http://192.168.1.5:8095`). Do not share the code. Restarting before completing setup generates a new code.

### 5. Complete the wizard

- Choose **Basic (Recommended)** unless you specifically need advanced settings.
- **Recommended for real installations:** Choose local accounts / sign-in required, and create an administrator account during the wizard. Choosing **no sign-in** finishes setup with no local accounts. The one-time setup cannot be reopened afterward, and adding an administrator later is not straightforward.
- Leave the internal server listener at `:8080`. Docker handles any different **Host Port**.
- For file storage, choose `/storage`, which is already mapped to your persistent host directory.
- Optional discovery and other integrations can be configured to suit your network.

After completing the wizard, **restart the Prusa Connect container**. Your configuration and uploaded files should persist through restarts.

### 6. Verify the new server

Check the browser UI and the container status in Unraid. From an Unraid terminal, using the **Host Port** you chose:

```bash
curl -fsS http://127.0.0.1:8080/api/v1/health
curl -fsS http://127.0.0.1:8080/api/v1/readiness
```

Replace `8080` in these **host-side curl commands** if you chose a different host port. A successful readiness response has `"ready":true`; after Basic setup, storage and history should become `ready`.

### If the container is restarting with "permission denied"

1. Stop **only** the Prusa Connect container in Unraid's Docker tab.
2. Review all five actual **host paths** in its Edit page.
3. Run the directory creation and `chown -R 10001:10001` commands for **those Prusa Connect directories only**.
4. Start the container again and reopen its logs for the setup code.

Do not delete the appdata or recreate the container just to fix this error.

## Configuration details

The writable settings directory is mounted at `/config/config.d`, consistent with [Prusa's container documentation](https://github.com/prusa3d/prusa-connect/blob/main/docs/src/content/docs/container.md). **An external `config.yaml` is not required** for a first installation: the Basic setup wizard generates persistent configuration and can initialize file storage at `/storage`.

If you later need a customized base configuration, bind-mount a readable host file **read-only** to `/config/config.yaml`. That is a separate file mapping, not the same path as the writable `/config/config.d` mount. Restart the application after configuration changes.

The template uses a read-only root filesystem, drops all Linux capabilities, enables `no-new-privileges`, and preserves the official image's non-root user.

### Existing installations

**Do not change a working older container's mount from `/config` to `/config/config.d` without planning a data migration.** Docker does not migrate the contents of an existing mount when its destination changes. New installations should use the updated layout.

## Image version and updates

| Setting | Template default |
|---|---|
| Image | `ghcr.io/prusa3d/prusa-connect:0.1.0-beta.1` |
| Host port | `8080` (editable during installation) |
| Container port | `8080/tcp` |
| Network | Bridge |
| Writable paths | `/config/config.d`, `/data`, `/files`, `/storage`, `/tmp` |

Prusa publishes prereleases under exact tags and does not move `latest` for beta versions. This template is temporarily pinned to the initial published beta so fresh installs can pull it. When Prusa releases a stable image with a `latest` tag, the template can switch to that tag for new installs. **Existing pinned installs must change the Repository field once** to start tracking that channel. Unraid can then detect new official image versions; this template does not automatically install updates.

## Support

- [Unraid template issues](https://github.com/bleugray/prusa-connect-unraid/issues)
- [Prusa Connect source](https://github.com/prusa3d/prusa-connect)
- [Prusa official container image](https://github.com/prusa3d/prusa-connect/pkgs/container/prusa-connect)
- [Prusa container documentation](https://github.com/prusa3d/prusa-connect/blob/main/docs/src/content/docs/container.md)

The application icon is referenced from Prusa's official repository and is not redistributed here.
