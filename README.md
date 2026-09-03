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

> Hyper-V fails to start VM saying : Start PXE over IPv4

That message means Hyper-V couldn't find a bootable device in the boot order and fell through to network boot (PXE) as the last resort. It's not actually trying to PXE-boot on purpose — it's telling you nothing else in the boot chain worked.

The official release for v4.0.2 only publishes a SHA256 checksum. 

- Official SHA256 (from the GitHub release): 2ef8e624aa1bec7e277e28056b8535a6c9373ba48d7ede3f1a01cb6d2373cfb8

```
mickm@ubuntu24-gemini:/mnt/c/Users/306270/local storage/BIG files/ISO$ sha256sum omarchy-4.0.2.iso
2ef8e624aa1bec7e277e28056b8535a6c9373ba48d7ede3f1a01cb6d2373cfb8  omarchy-4.0.2.iso
mickm@ubuntu24-gemini:/mnt/c/Users/306270/local storage/BIG files/ISO$
```

That checksum matches exactly — your ISO is genuine and not corrupted, so that rules out a bad download as the cause of the earlier PXE boot issue.

> I switched to using Gen 1 VM in Hyper-V and that seems to be the solution.

Interesting that Gen 1 fixed it. That actually makes sense in hindsight for a WSL2/Hyper-V setup like yours — a few likely reasons Gen 2 was failing:

- **DVD drive controller mismatch** – if the ISO got attached to an IDE controller instead of SCSI (easy to do by habit, since Gen 1 defaults to IDE), a Gen 2 VM won't boot it at all and falls through to PXE. Gen 1 just uses IDE natively, so it worked without needing to think about it.
- **Secure Boot/firmware quirks** – some Hyper-V builds still have inconsistent UEFI firmware behavior with certain Linux ISOs, especially ones like Omarchy's Arch-based installer that don't ship Secure Boot signed bootloaders. Gen 1's legacy BIOS path sidesteps that whole layer.
- **Boot order state** – Gen 1's simpler boot order (floppy/CD/IDE/network) is less finicky than Gen 2's UEFI boot entries, which can get out of sync if the VM was created and reconfigured a few times.

Good outcome either way. Only trade-off to keep in mind with Gen 1: you're on legacy BIOS/MBR now, and disk will be capped at 2TB and boot slightly slower than Gen 2 would — but for a lab/desktop VM that's rarely going to matter.

> Hyper-V install took 13 mins ... is that normal ?

Yes, that's normal — actually on the faster side for a Gen 1 VM.

A few things affect Omarchy install time:

- **Gen 1 uses emulated IDE, not synthetic drivers** — this is the big one for you. Since you switched from Gen 2 to Gen 1 to fix the boot issue, you lost Hyper-V's synthetic (VMBus) disk and network drivers. Gen 1 falls back to legacy emulated IDE controllers, which are noticeably slower for disk I/O — this alone can turn a 5-minute install into 10-15 minutes.
- **Host disk speed** — you're installing from an ISO sitting on `/mnt/c/...` (Windows filesystem via WSL2 path), so there's a layer of translation happening. If the underlying disk is HDD rather than NVMe/SSD, that adds real time.
- **Recent Omarchy releases have optimized for speed** — the 3.8.3 changelog specifically called out installs shrinking the ISO and speeding up installation by +30%, with sub-minute installs possible on fast NVMe setups. That's the best-case scenario though, and it assumes synthetic/virtio-class drivers, not legacy IDE emulation.

So 13 minutes on Gen 1 with emulated IDE lines up with expectations — you're trading some install/runtime speed for the boot reliability. If install speed becomes annoying (e.g., you're reinstalling often for testing), one option is to try Gen 2 again but explicitly force the ISO onto the SCSI controller and double check boot order via PowerShell, which was likely the actual root cause of your original PXE issue rather than Gen 2 itself. But if Gen 1 is working and you're not reinstalling repeatedly, no need to touch it.

> Tried to fix resolution in Hyper-V viewer ... gave up after spending too much time messing around.

## Display lessons 

- Don't use MremoteNG because it's VNC support is out of date - use TigerVNC or RealVNC viewers instead
- Used wayvnc on omarchy because of xorg support ???
- Omarchy doesn't ship vi/nano, append via heredoc rather than opening an editor.
