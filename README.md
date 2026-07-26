## ❓ About kernel-actions (or my personal nickname for it, lazykernel.)
**This is a simple project where I compile the Linux kernel on Github's Actions because I wanted to have linux-tkg style performance while not building it myself, hence that's why I call it lazykernel.**

![Latest Commit](https://img.shields.io/github/last-commit/dozingyobs/kernel-actions)
> [!NOTE]
> 07/26/2026 today i just realized tkg already has a github actions pipeline and a prebuilt kernel in the first place, which uh kind of makes this project a whole lot useless i think

## 🧭 Quick Links
* 📦 ⬇️ [Jump to Installation](#install)
* 📦 ⬆️ [Jump to maintenance/uninstall kernel](#uninstall)
* 🖧 [(**EXPERIMENTAL**) Jump to Server Kernel](#server-kernel)
* 📊 [Jump to Hardware used and Benchmark Results](#benchmarks)

## ℹ️ Some info you might wanna know

> [!IMPORTANT]
> * **Stable Track:** Uses the latest upstream Stable source (7.1.y at the moment)
> * **Longterm Track:** Uses the latest upstream LTS source (6.18.y at the moment)
> * **Automation:** Compiled automatically via GitHub Actions on every release.
> * **Secure Boot:** **Must be disabled**. Leaving Secure Boot active will prevent your system from booting this kernel.

> [!WARNING]  
> **NVIDIA & External Driver Modules:** Because this kernel is compiled using **Clang/LLVM with ThinLTO**, any out-of-tree kernel modules (like proprietary NVIDIA drivers) **must** be compiled using the exact same Clang toolchain. Building drivers with standard GCC will fail. Before installing your graphics drivers, ensure you have the required tools installed on your host system:
> ```bash
> sudo apt update && sudo apt install build-essential dkms clang lld llvm libelf-dev
> ```

> [!WARNING]
> NVIDIA proprietary drivers **older than 580.x** will fail to build the `nvidia-drm` kernel module via DKMS against LazyKernel kernels ≥6.17 (currently 6.18.y, which is ≥6.17), with an error like:
> ```bash
>    error: incompatible function pointer types initializing
>    'struct drm_framebuffer *(*)(...)' with an expression of type
>    'struct drm_framebuffer *(...)'
> ```
> If you're on a **Maxwell/Pascal/Volta GPU**, use driver branch **580.x** (the final branch supporting those architectures). **Turing** and newer can use any **current** driver. Driver **470.x (Kepler)** and earlier are not expected to work
without manual patching.


## ⚙️ Features
- **BORE** scheduler for improved responsiveness
- **NTSYNC** module for improved Wine/Proton performance
- **1000Hz** tick rate with **NO_HZ_IDLE** for lower latency
- **BBRv3** TCP congestion control with **fq** queueing
- **Clang/LLVM** compiled with ThinLTO
- **x86-64-v3** optimizations
- Easy installation via a prebuilt .deb package

## 📋 Requirements
- **x86-64-v3** compatible CPU (basically a computer from 2015+)
- **Debian based distro.**
- A backup kernel (Recommended)

## ⚠️ Who is this for?
This kernel is intended for experienced Linux users who are comfortable with:
- Installing custom kernels manually
- Having secure boot disabled
- Recovering from a broken boot via GRUB or a live USB

If you're running a standard Ubuntu/Mint/Pop installation and don't know what **ANY** of these mean, stick with your distro's kernel. A bad kernel install can leave your system unbootable.

If you don't care then by all means go ahead and install it, however I am **NOT** responsible if your system ends up being unbootable lol

## 📦 ⬇️ How to install the prebuilt kernel? <a name="install"></a>
---
### Option A: Automated Installation via APT Repository
Automatic updates — no manual `.deb` downloads needed each release.

LazyKernel is published as **three independent tracks**. Add only the source(s) you want — you can enable more than one at a time (e.g. `stable` and `server` on different machines, or `stable` + `longterm` side-by-side on the same desktop if you want to dual-boot between them).

| Track | What it tracks | Best for |
|---|---|---|
| `stable` | Latest mainline **stable** kernel (currently 7.1.x) | Desktop / general use, newest features |
| `longterm` | Pinned **6.18.x LTS** branch | Desktop users who want a longer-lived, less-churny base |
| `server` | Power/efficiency-tuned server build | Headless home server — see 🖧 LazyKernel Server (Experimental) first |

#### 1. Import the Signing Key
```bash
curl -fsSL https://dozingyobs.github.io/kernel-actions/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kernel-actions.gpg
```

#### 2. Add the Track(s) You Want
**Desktop — stable:**
```bash
echo "deb [signed-by=/usr/share/keyrings/kernel-actions.gpg] https://dozingyobs.github.io/kernel-actions/stable ./" | sudo tee /etc/apt/sources.list.d/kernel-actions-stable.list
```
**Desktop — longterm (6.18.x LTS):**
```bash
echo "deb [signed-by=/usr/share/keyrings/kernel-actions.gpg] https://dozingyobs.github.io/kernel-actions/longterm ./" | sudo tee /etc/apt/sources.list.d/kernel-actions-longterm.list
```
**Server:**
```bash
echo "deb [signed-by=/usr/share/keyrings/kernel-actions.gpg] https://dozingyobs.github.io/kernel-actions/server ./" | sudo tee /etc/apt/sources.list.d/kernel-actions-server.list
```

#### 3. Install the Meta-Package for Your Track
```bash
sudo apt update
```
**Stable:**
```bash
sudo apt install linux-image-lazy-stable-meta
```
**Longterm:**
```bash
sudo apt install linux-image-lazy-longterm-meta
```
**Server:**
```bash
sudo apt install linux-image-lazy-server-meta
```
```bash
sudo apt upgrade
```
Each meta-package tracks the latest build for its own track independently, so future `apt upgrade` runs pull new kernels automatically. Old kernel versions are pruned automatically too (current + 1 previous kept, per track).

#### 4. Reboot & Verify
```bash
sudo reboot
uname -r
```
---
### Option B: Manual Installation
If you'd rather download and install `.deb` files by hand each release:

**1. Download** both `linux-image-*-lazy_*_amd64.deb` and `linux-headers-*-lazy_*_amd64.deb` for your track from the **Releases** tab — stable builds are tagged `*-stable-debug`, longterm builds `*-longterm-debug`, server builds carry no moniker.

**2. Install:**
```bash
sudo dpkg -i linux-image-*.deb linux-headers-*.deb
```
**3. Reboot & Verify:**
```bash
sudo reboot
uname -r
```
---
## 📦 ⬆️ Basic Maintenance / Uninstalling Older Kernels <a name="uninstall"></a>
---
> [!NOTE]
> If you installed via the **APT repository** (Option A), old kernels are pruned automatically per track — you shouldn't need this section under normal use. It's here for manual installs (Option B) or if you want to force a cleanup.

> [!NOTE]
> Both `stable` and `longterm` desktop builds use the same `-lazy` package suffix (only the version number differs — e.g. `7.1.5-lazy` vs `6.18.40-lazy`), so if you have both tracks enabled, `dpkg -l | grep lazy` will list kernels from both together. Check the version number against which track you meant to prune.

**1. List installed LazyKernel packages:**
```bash
dpkg -l | grep lazy
```
**2. Purge the old/unwanted version** (double-check it does NOT match your active `uname -r`):
```bash
sudo apt purge linux-image-[OLD-VERSION]-lazy linux-headers-[OLD-VERSION]-lazy
```
**Example:** if `dpkg -l | grep lazy` shows:
```text
ii  linux-image-6.18.30-lazy   6.18.30-1   amd64   Linux kernel binary image
ii  linux-image-6.18.35-lazy   6.18.35-1   amd64   Linux kernel binary image
```
and your active kernel is `6.18.35-lazy`, remove the older one:
```bash
sudo apt purge linux-image-6.18.30-lazy linux-headers-6.18.30-lazy
```
If you see a warning like:
```
dpkg: warning: while removing linux-image-6.18.38-lazy, directory '/lib/modules/6.18.38-lazy' not empty so not removed
```
clean up the leftover module directory manually (~70MB each):
```bash
sudo rm -rf /lib/modules/6.18.38-lazy
```
**3. Refresh GRUB** so removed kernels disappear from the boot menu:
```bash
sudo update-grub
```
---
## 🖧 LazyKernel Server (Experimental) <a name="server-kernel"></a>
---

> [!NOTE]
> This is a **separate build** from the desktop LazyKernel above, tuned specifically for headless server workloads (file serving, self-hosted apps, etc.) rather than desktop responsiveness or gaming. If you're setting up a desktop or gaming machine, use the [regular install](#install) instead.

> [!WARNING]
> **HEADS UP!**
> I do **NOT** recommend installing this kernel (kernels from a server are meant to be boring and stable, not something to be "optimized") as I've slashed unnecessary kernel features and modules that affect compatibility or behavior you depend on.
>
> I advise you to have a backup kernel by any means necessary. If you install this, it's your responsibility now — not mine.

> [!WARNING]
> This mostly favors **Intel CPUs**, especially the **i3-8145U** (the exact hardware this build is tuned and tested on). If you're on AMD hardware, the config options have been added however they have not been tested; CPU-agnostic tuning (BBRv3, HZ/tick rate, preemption model, module signing) should still work fine regardless of vendor.

### ⚙️ Server Features
- **BBRv3** TCP congestion control with **fq** queueing (shared with desktop build)
- **100Hz** tick rate with **NO_HZ_IDLE** — lower timer overhead, better idle power draw
- **VOLUNTARY** preemption — favors power efficiency and throughput over desktop-style low-latency responsiveness
- **schedutil** CPU frequency governor + Intel P-State/idle support — tuned for lower power draw on always-on hardware
- Module signing disabled (for home servers)
- Debug info stripped for smaller image size and faster builds
- **x86-64-v3** optimizations, Clang/LLVM ThinLTO (same toolchain requirements as desktop — see the NVIDIA/DKMS warning above if applicable)
- Stripped input subsystems irrelevant to a headless box: no mouse, joystick, tablet, or touchscreen drivers
- No sound subsystem (`CONFIG_SOUND` disabled) — this machine will **NOT** have audio
- No Bluetooth stack
- Legacy/discrete-GPU framebuffer drivers removed (kept modern DRM/KMS only, since the iGPU still needs it for console output)
- Staging (experimental) drivers disabled

### 📋 Requirements
- **x86-64-v3** compatible CPU
- **Debian based distro** (tested against Debian 13 Trixie, headless)
- A backup kernel (**strongly recommended** — this is experimental and less battle-tested than the desktop build)

### 📦 How to Install

**Via APT** (see [Option A](#install) above) — use `linux-image-lazy-server-meta` instead of the desktop meta package.

**Manual install:**
Download `linux-image-*-lazyserver_*.deb` and `linux-headers-*-lazyserver_*.deb` from the **Releases** tab (tagged `-server`):
```bash
sudo dpkg -i linux-image-*-lazyserver*.deb linux-headers-*-lazyserver*.deb
sudo reboot
```

Verify:
```bash
uname -r
```
---
### 🖥️ Test Hardware Configuration <a name="benchmarks"></a>
Before looking at the benchmark data below, you can review the exact machine specs used to run these performance tests:

<details>
<summary><b>Click to expand hardware</b></summary>

| Component | Specification |
| :--- | :--- |
| **CPU** | Intel Core i7-7700HQ @ 2.80GHz (4 Cores / 8 Threads) |
| **RAM** | 16GB DDR4 @ 2400MHz |
| **GPU** | NVIDIA GeForce GTX 1050 Mobile |
| **Storage** | 500GB Samsung 970 EVO NVMe SSD |
| **OS / Distro** | Debian 13 (Trixie) |

</details>

---

## 📊 Benchmark results
<img width="373" height="386" alt="Screenshot 2026-06-17 at 06-50-35 kernel-benchmarks - Phoronix Test Suite" src="https://github.com/user-attachments/assets/f8d9b8e3-325b-4b82-bdf6-d273f744dad2" />
<img width="867" height="399" alt="Screenshot 2026-06-17 at 06-49-19 kernel-benchmarks - Phoronix Test Suite" src="https://github.com/user-attachments/assets/8a96ff58-1209-4947-a694-66ceb0eabc64" />
<img width="867" height="348" alt="Screenshot 2026-06-17 at 06-50-48 kernel-benchmarks - Phoronix Test Suite" src="https://github.com/user-attachments/assets/2a57c7e8-fe9e-4ccf-ac1f-d5c2540143a5" />
<img width="867" height="399" alt="Screenshot 2026-06-17 at 06-50-44 kernel-benchmarks - Phoronix Test Suite" src="https://github.com/user-attachments/assets/f05351e0-4f00-47bf-bdde-1e3c7048d66e" />



