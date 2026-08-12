# Custom ISO deployment

This repository can install a locally supplied Windows ISO without adding the ISO or the resulting VM disk to Git. The reusable example is `compose.iot-ltsc.yml`.

## Requirements

- Linux host with Docker Compose and `/dev/kvm`.
- A licensed, bootable Windows ISO supplied by the operator.
- Sufficient RAM and disk space for the VM.
- Separate storage directory for every VM instance.

Docker Desktop on macOS does not normally expose KVM to this container. Use a Linux host or a VM with nested virtualization.

## First-time setup

```bash
cp .env.example .env
chmod 600 .env
# Edit .env: set ISO_PATH, STORAGE_PATH, ports, and WINDOWS_PASSWORD.
mkdir -p oem
cp oem/install.bat.example oem/install.bat

docker compose -f compose.iot-ltsc.yml config
docker compose -f compose.iot-ltsc.yml up -d
docker compose -f compose.iot-ltsc.yml logs -f
```

When `/custom.iso` is mounted, the `VERSION` value is not used to choose the media. The ISO is detected by the container and installed into the persistent `/storage` disk.

The web console is published on `WEB_PORT`; RDP is published on `RDP_PORT`. Change these if another service already uses the defaults.

## Customization

Files mounted from `oem/` are available under `C:\\OEM`. If `oem/install.bat` exists, it is run during the final installation stage. Keep repeatable setup scripts in the repository, but keep proprietary installers, credentials, ISO files, and VM disks outside Git.

## Reusable VM templates

After the VM is configured, stop it cleanly and copy its storage directory to a new location. Never run two VMs against the same storage directory.

```bash
docker compose -f compose.iot-ltsc.yml down
cp -a /path/to/template-storage /path/to/windows-dev-01
```

For cloned machines that will coexist on a network, use Sysprep or another supported Windows generalization process so they receive unique machine identity and network settings.

## Security and licensing

- `.env` is ignored and must never be committed.
- Do not commit Windows product keys, passwords, ISO files, or VM disks.
- Restrict public web-console and RDP ports with a firewall or use an SSH tunnel on non-disposable hosts.
- Use installation media and product keys in accordance with Microsoft's licensing terms.
