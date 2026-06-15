## ❓ About kernel-actions (or my personal nickname for it, lazykernel.)
**This is a simple project where I compile the Linux kernel on Github's Actions because I wanted to have linux-tkg style performance while not building it myself, hence that's why I call it lazykernel.**

## ℹ️ Some info you might wanna know
- Uses the most latest LTS source (6.18.y at the moment)
- Secure Boot must be disabled — modules are unsigned
- Built automatically via GitHub Actions on every release

## ⚙️ Features
- **BORE** scheduler for improved responsiveness
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

## 📦 ⬇️ How to install the prebuilt kernel?
---

### Step 1: Download the Releases
Go to the **Releases** tab on GitHub and download both `linux-image-6.x.x-bore_6.x.x-x_amd64.deb` AND `linux-headers-6.x.x-bore_6.x.x-x_amd64.deb` files to your local machine.

---

### Step 2: Install the Packets
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

If you're running a standard Ubuntu/Mint/Pop installation and don't know what **ANY** of these mean, stick with your distro's kernel. A bad kernel install can leave your system unbootable.

If you dont care then by all means go ahead and install it, however I am **NOT** responsible if your system ends up being unbootable lol
