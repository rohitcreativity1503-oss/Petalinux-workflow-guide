# PetaLinux Workflow Guide

This repository contains a step-by-step guide for creating, configuring, building, packaging, and deploying a PetaLinux project on Zynq-based platforms.

---

# Table of Contents

- [Quick Navigation](#quick-navigation)
- [Create a PetaLinux Project](#create-a-petalinux-project)
- [Attach Vivado Hardware Design (XSA)](#attach-vivado-hardware-design-xsa)
- [Configure Root Filesystem Packages](#configure-root-filesystem-packages)
- [Configure Linux Kernel](#configure-linux-kernel)
- [Configure U-Boot](#configure-u-boot)
- [Build the Project](#build-the-project)
- [Generate BOOT.BIN](#generate-bootbin)
- [Create SD Card Image](#create-sd-card-image)
- [PetaLinux Application Development](#petalinux-application-development)
  - [Create an Application](#create-an-application)
  - [Application Location](#application-location)
  - [Adding Additional Source Files](#adding-additional-source-files)
  - [Build Application](#build-application)
- [PetaLinux SDK Generation](#petalinux-sdk-generation)
  - [Build SDK](#build-sdk)
  - [Extract SDK](#extract-sdk)
  - [Generate Sysroot](#generate-sysroot)
  - [Source SDK Environment](#source-sdk-environment)
  - [Cross-Compile an Application](#cross-compile-an-application)
- [Typical PetaLinux Build Flow](#typical-petalinux-build-flow)
- [Useful Commands](#useful-commands)

---

# Quick Navigation

| Task | Link |
|--------|--------|
| Create New Project | [Go](#create-a-petalinux-project) |
| Import XSA | [Go](#attach-vivado-hardware-design-xsa) |
| Configure RootFS | [Go](#configure-root-filesystem-packages) |
| Configure Kernel | [Go](#configure-linux-kernel) |
| Configure U-Boot | [Go](#configure-u-boot) |
| Build Linux Project | [Go](#build-the-project) |
| Generate BOOT.BIN | [Go](#generate-bootbin) |
| Create SD Card Image | [Go](#create-sd-card-image) |
| Create Application | [Go](#create-an-application) |
| Build SDK | [Go](#build-sdk) |
| Cross Compile Application | [Go](#cross-compile-an-application) |

---

# Create a PetaLinux Project

## Create a New Project

```bash
petalinux-create project --template zynq --name linux_os
```

## Create a Project from BSP

```bash
petalinux-create project -s <path_to_bsp_file>
```

Move into the project directory:

```bash
cd linux_os
```

---

# Attach Vivado Hardware Design (XSA)

Import the exported XSA file:

```bash
petalinux-config --get-hw-description <path_to_xsa_file>
```

## Required Configuration

### Root Filesystem Type

Set:

```text
Root Filesystem Type -> EXT4 (SD/eMMC/SATA/USB)
```

### Disable TFTP Boot Copy

Navigate to:

```text
Image Packaging Configuration
```

Disable:

```text
Copy final images to TFTP boot directory
```

Re-open project configuration if needed:

```bash
petalinux-config
```

---

# Configure Root Filesystem Packages

Add required packages such as:

- Python
- Net-tools
- OpenCV
- SSH
- Custom packages

```bash
petalinux-config -c rootfs
```

---

# Configure Linux Kernel

```bash
petalinux-config -c kernel
```

---

# Configure U-Boot

```bash
petalinux-config -c u-boot
```

---

# Build the Project

Build the complete PetaLinux project:

```bash
petalinux-build
```

---

# Generate BOOT.BIN

```bash
petalinux-package --boot \
--fsbl ./images/linux/zynq_fsbl.elf \
--fpga ./images/linux/system.bit \
--u-boot
```

Generated file:

```text
BOOT.BIN
```

---

# Create SD Card Image

## Option 1: Generate WIC Image

```bash
petalinux-package --wic \
--bootfiles "BOOT.BIN image.ub system.dtb boot.scr" \
--rootfs-file ./images/linux/rootfs.tar.gz
```

---

## Option 2: Manual SD Card Preparation

### Partition 1

Format:

```text
FAT32
```

Recommended Size:

```text
1 GB
```

Copy:

```text
BOOT.BIN
image.ub
system.dtb
boot.scr
```

### Partition 2

Format:

```text
EXT4
```

Recommended Size:

```text
15 GB or larger
```

Extract:

```bash
rootfs.tar.gz
```

to the EXT4 partition.

---

# PetaLinux Application Development

## Create an Application

Example:

```bash
petalinux-create apps --template c --name linux-hello-world --enable
```

---

## Application Location

```text
<PetaLinux_Project>/project-spec/meta-user/recipes-apps/
```

Example:

```text
project-spec/
└── meta-user/
    └── recipes-apps/
        └── linux-hello-world/
```

---

## Adding Additional Source Files

If additional source/header files are copied into the application's files directory:

```text
linux-hello-world/
├── files/
│   ├── linux-hello-world.c
│   ├── helper.c
│   └── helper.h
└── linux-hello-world.bb
```

Update the BitBake recipe:

```bitbake
SRC_URI += "file://helper.c"
SRC_URI += "file://helper.h"
```

---

## Build Application

Build the complete project:

```bash
petalinux-build
```

---

# PetaLinux SDK Generation

## Build SDK

Generate SDK files:

```bash
petalinux-build --sdk
```

---

## Extract SDK

Run:

```bash
./images/linux/sdk.sh
```

Follow the installation prompts.

---

## Generate Sysroot

```bash
petalinux-package --sysroot
```

---

## Source SDK Environment

Example generated environment files:

### Project SDK

```bash
source /home/rohit/project_petalinux/linux_os/images/linux/sdk/environment-setup-cortexa9t2hf-neon-amd-linux-gnueabi
```

### Installed SDK

```bash
source /opt/petalinux/2025.2/environment-setup-cortexa9t2hf-neon-amd-linux-gnueabi
```

Source both before cross-compiling.

---

## Cross-Compile an Application

Example:

```bash
$CC onn.c -o app
```

Verify target architecture:

```bash
file app
```

Expected output:

```text
ELF 32-bit LSB executable, ARM...
```

---

# Typical PetaLinux Build Flow

```text
Create Project
      │
      ▼
Import XSA
      │
      ▼
Configure RootFS
      │
      ▼
Configure Kernel
      │
      ▼
Configure U-Boot
      │
      ▼
Build Project
      │
      ▼
Generate BOOT.BIN
      │
      ▼
Create SD Card Image
      │
      ▼
Boot Target Board
      │
      ▼
Generate SDK (Optional)
      │
      ▼
Cross-Compile Applications
```

---

# Useful Commands

## Open Project Configuration

```bash
petalinux-config
```

## Configure RootFS

```bash
petalinux-config -c rootfs
```

## Configure Kernel

```bash
petalinux-config -c kernel
```

## Configure U-Boot

```bash
petalinux-config -c u-boot
```

## Build Project

```bash
petalinux-build
```

## Build SDK

```bash
petalinux-build --sdk
```

## Clean RootFS

```bash
petalinux-build -c rootfs -x cleanall
```

## Clean Entire Project

```bash
petalinux-build -x mrproper
```

## Generate BOOT.BIN

```bash
petalinux-package --boot \
--fsbl ./images/linux/zynq_fsbl.elf \
--fpga ./images/linux/system.bit \
--u-boot
```

## Generate WIC Image

```bash
petalinux-package --wic \
--bootfiles "BOOT.BIN image.ub system.dtb boot.scr" \
--rootfs-file ./images/linux/rootfs.tar.gz
```

---

# Author

**Rohit Gupta**  
M.Tech, Integrated Circuit Design and Technology  
IIT Gandhinagar
