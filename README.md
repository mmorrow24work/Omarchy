# Omarchy
Omarchy is an omakase Linux distribution based on Arch, the tiling window manager Hyprland, and the desktop construction-kit Quickshell. It ships with everything a modern, savvy computer user needs to be productive immediately. From Neovim (btw) to Chromium, Obsidian to LibreOffice, and Kdenlive to OBS Studio.

https://omarchy.org

# VM install

> omarchy-4.0.2 ... when i create a VM using Hyper-V - do i use gen 1 or 2 ?

Gen 2, for Omarchy (and Arch-based distros generally).

Why Gen 2 is the right call:
- **UEFI boot** – Omarchy's installer (like most modern Arch-based ISOs) expects UEFI, and Gen 2 VMs boot via UEFI natively. Gen 1 is legacy BIOS only, which can cause the installer to behave oddly or fail to boot the ISO properly.
- **Faster boot and better performance** – Gen 2 uses synthetic drivers throughout (no legacy emulated devices), so disk and network I/O are noticeably quicker.
- **GPT disks, >2TB support** – not usually a big deal for a desktop-style install, but nice to have.

One thing to configure explicitly: **disable Secure Boot** on the Gen 2 VM's firmware settings (or set the Secure Boot template to "Microsoft UEFI Certificate Authority" if you want to try keeping it on — but Arch/Omarchy isn't signed for Secure Boot out of the box, so disabling is the safe default). If you skip this, the VM will typically fail to boot the ISO or the installed system.

Quick setup checklist:
1. New-VM with `-Generation 2`
2. In VM Settings → Security, untick "Enable Secure Boot" (or set template as above)
3. Firmware → set DVD drive first in boot order, attach the Omarchy ISO
4. Give it at least 4GB RAM (8GB+ recommended, Omarchy runs Hyprland/Wayland which likes some headroom) and 2+ vCPUs
5. If you plan to use it as a desktop with GPU accel, note Hyper-V's synthetic GPU (RemoteFX successor) support is limited — enhanced session mode over RDP is usually the smoother path for a Wayland compositor like Hyprland rather than the Basic Session VMConnect console.

If you hit boot issues after installation, double-check the disk was partitioned as GPT (default for UEFI installs) — Gen 2 won't boot an MBR disk.
