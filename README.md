# Xiaomi AX6000 (RA72): LuCI + DoH installer bundle

This bundle reproduces the **data-only overlay** currently running on the AX6000. It adds LuCI and a Cloudflare DNS-over-HTTPS proxy while leaving the target router's Xiaomi firmware, kernel, bootloader, Wi-Fi drivers, calibration data, and flash partitions in place. It does not copy Wi-Fi credentials, WAN settings, root password, SSH keys, or device-specific data; those remain unique to each router.

It is **not** a `sysupgrade.bin` or recovery image. The current router is Xiaomi firmware `1.0.122`, reporting OpenWrt `18.06-SNAPSHOT`, target `ipq/ipq50xx_64`, board `ap-mp03.1`, architecture `aarch64_cortex-a53`, and kernel `4.4.60`. The installer checks the board, target, architecture, release, and kernel before writing. Use it only on another AX6000/RA72 with the same firmware family and version.

## Before installing on another router

1. Use the Xiaomi router patcher separately to obtain root access on that router.
2. Set up a **new, device-specific SSH key** and confirm that `ssh root@ROUTER_IP` works with that key. This bundle contains no SSH private key and does not open root access.
3. Keep the new router reachable over LAN or Wi-Fi while the install runs. The installer briefly reloads the firewall.

On a Mac, run:

```sh
./install.command ROUTER_IP /absolute/path/to/that-router-root-key
```

The SSH key must already authenticate as root. The installer does not contain or change the target's passwords. LuCI uses that router's Linux root password; set a strong one with `passwd` over the root SSH session if needed.

## What it installs

- LuCI at `http://ROUTER_IP/`, including a **Services → DNS HTTPS Proxy** page.
- `https-dns-proxy` listening only on `127.0.0.1:5053`, using Cloudflare DoH.
- dnsmasq forwarding to that local proxy, with its ordinary resolv-file upstream disabled.
- A firewall startup hook for the LuCI overlay and the DoH service.
- A per-device rollback backup under `/data/ax6000-clean-mod/rollback/`.

Clients that use the router as their DNS server receive DoH-backed answers. The bundle does not redirect devices that manually use an external DNS server.

## Roll back

From the Mac, run:

```sh
./rollback.command ROUTER_IP /absolute/path/to/that-router-root-key
```

Or connect over SSH and run:

```sh
sh /data/ax6000-clean-mod/rollback_stock_ui.sh
```

Rollback restores the saved firewall, system, DHCP, and LuCI settings, stops the DoH proxy, and re-enables the Xiaomi web services. It does not restore the target's root password or remove the SSH key you installed for that target.

## Compatibility and provenance

The DoH executable and LuCI app are pinned to official OpenWrt 19.07 packages for `aarch64_cortex-a53`; their required libraries were present on the AX6000 firmware used to build this bundle. The LuCI base overlay was assembled from the official OpenWrt 18.06.9 package set. Package archives, source locations, and SHA-256 digests are recorded in `payload/sources/SOURCES.md` and retained in this bundle. The DoH LuCI page contains small compatibility adjustments for this data-only overlay.

The base system is still the old Xiaomi/OpenWrt 18.06 snapshot. This bundle does not update that firmware or its kernel.
