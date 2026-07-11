## ❓ About kernel-actions (or my personal nickname for it, lazykernel.)
**This is a simple project where I compile the Linux kernel on Github's Actions because I wanted to have linux-tkg style performance while not building it myself, hence that's why I call it lazykernel.**

![Latest Commit](https://img.shields.io/github/last-commit/dozingyobs/kernel-actions)

## 🧭 Quick Links
* 📦 ⬇️ [Jump to Installation](#install)
* 📦 ⬆️ [Jump to maintenance/uninstall kernel](#uninstall)
* 🖧 [(**EXPERIMENTAL**) Jump to Server Kernel](#server-kernel)
* 📊 [Jump to Hardware used and Benchmark Results](#benchmarks)

## ℹ️ Some info you might wanna know

> [!IMPORTANT]
> * **Active Branch:** Uses the latest upstream LTS source (6.18.y at the moment)
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

### Step 1: Download the Releases
Go to the **Releases** tab on GitHub and download both `linux-image-6.x.x-lazy_6.x.x-x_amd64.deb` AND `linux-headers-6.x.x-lazy_6.x.x-x_amd64.deb` files to your local machine.

---

### Step 2: Install the Packages
Open your terminal, navigate to your download folder, and run `dpkg` to install both packages simultaneously:

```bash
sudo dpkg -i linux-image-6.*.deb linux-headers-6.*.deb
```
### Step 3: Reboot & Verify
Reboot your machine to boot into the newly installed kernel:

```bash
sudo reboot
```
Once your system restarts, verify the installation by checking the active kernel version:
```bash
uname -r
```
### Automated Installation via APT Repository (Experimental)

If you prefer automated updates instead of downloading `.deb` files manually every time, you can add the custom repository to your system.

#### 1. Add the Repository Source
Run the following command to add the repository to your APT sources list:
```bash
curl -fsSL https://dozingyobs.github.io/kernel-actions/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kernel-actions.gpg
echo "deb [signed-by=/usr/share/keyrings/kernel-actions.gpg] https://dozingyobs.github.io/kernel-actions/debs ./" | sudo tee /etc/apt/sources.list.d/kernel-actions.list
```

#### 2. Update and Install
Refresh your package lists and install the kernel packages using `apt`:
```bash
sudo apt update
sudo apt install linux-image-lazy-meta
sudo apt upgrade
```

#### 3. Reboot & Verify
```bash
sudo reboot
```
Once your system restarts, verify that the kernel is active:
```bash
uname -r
```

## 📦 ⬆️ How to do basic maintenance or uninstall older versions of the kernel <a name="uninstall"></a>
---

### Step 1: List all installed lazykernel (or lazy) packages
Because these are manual installations, your system will NOT automatically remove older versions. Open your terminal and run this command to find your installed packages:

```bash
dpkg -l | grep lazy
```

---

### Step 2: Purge the old or unwanted kernels
Copy the exact name of the older package from the list (making sure it does NOT match your active `uname -r` version) and completely remove it using `apt purge`:

```bash
sudo apt purge linux-image-[PASTE-OLD-VERSION-HERE]-lazy linux-headers-[OLD-VERSION]-lazy
```

#### 💡 Example:
If running `dpkg -l | grep lazy` shows this output:
```text
ii  linux-image-6.18.30-lazy   6.18.30-1   amd64   Linux kernel binary image
ii  linux-image-6.18.35-lazy   6.18.35-1   amd64   Linux kernel binary image
```
And your current active kernel is `6.18.35-lazy`, you would remove the older `6.18.30` version by running:
```bash
sudo apt purge linux-image-6.18.30-lazy linux-headers-6.18.30-lazy
```

---

### Step 3: Update your boot menu
Once the package is removed, you must refresh your GRUB configuration file so the old options disappear from your system's boot selection screen:

```bash
sudo update-grub
```

## 🖧 LazyKernel Server (Experimental) <a name="server-kernel"></a>
---

> [!NOTE]
> This is a **separate build** from the desktop LazyKernel above, tuned specifically for headless/server workloads (file serving, self-hosted apps, etc.) rather than desktop responsiveness or gaming. If you're setting up a desktop or gaming machine, use the [regular install](#install) instead.

> [!WARNING]
> **HEADS UP!**
> I do **NOT** recommend installing this kernel (kernels from a server are meant to be boring and stable, not something to be "optimized") as I've slashed unnecessary kernel features and modules that affect compatibility or behavior you depend on.
> 
> I advise you to have a backup kernel by any means necessary. If you install this, it's your responsibility now — not mine.


> [!WARNING]
> This mostly favors **Intel CPUs**, especially the **i3-8145U** (the exact hardware this build is tuned and tested on). AMD-specific power-management options (`amd_pstate`, etc.) are **not included** in this config — not because they'd conflict with the Intel ones, but simply because they haven't been tested. If you're on AMD hardware, the config options have been added however they have not been tested; CPU-agnostic tuning (BBRv3, HZ/tick rate, preemption model, module signing) should still work fine regardless of vendor.


### ⚙️ Server Features
- **BBRv3** TCP congestion control with **fq** queueing (shared with desktop build)
- **100Hz** tick rate with **NO_HZ_IDLE** — lower timer overhead, better idle power draw
- **VOLUNTARY** preemption — favors power efficiency and throughput over desktop-style low-latency responsiveness
- **schedutil** CPU frequency governor + Intel P-State/idle support — tuned for lower power draw on always-on hardware
- Module signing disabled (for home servers)
- Debug info stripped for smaller image size and faster builds
- **x86-64-v3** optimizations, Clang/LLVM ThinLTO (same toolchain requirements as desktop — see the NVIDIA/DKMS warning above if applicable)

### 📋 Requirements
- **x86-64-v3** compatible CPU
- **Debian based distro** (tested against Debian 13 Trixie, headless)
- A backup kernel (**strongly recommended** — this is experimental and less battle-tested than the desktop build)

### 📦 How to install
Download `linux-image-*-lazyserver_*.deb` and `linux-headers-*-lazyserver_*.deb` from the **Releases** tab (tagged `-debug`, currently manual/draft releases only — no APT repo yet for the server build):

```bash
sudo dpkg -i linux-image-*-lazyserver*.deb linux-headers-*-lazyserver*.deb
sudo reboot
```

Verify:
```bash
uname -r
```

> [!WARNING]
> There is currently **no APT repository** for the server build — it's manual `dpkg -i` only per release. This may change in the future if there's demand for automatic updates.
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




