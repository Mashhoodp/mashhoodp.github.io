---
title: Installing Windows Server 2022 in VMware Workstation
description: This guide will show you how to install and configure Windows Server 2022 for AD vulnerability testing
author: mashhood
date: 2024-12-18 10:32:00 +0800
categories: [System Security]
tags: [windows] [server]
media_subpath: '/assets/img/posts/20241218/'
---
## Installing Windows Server 2022 in VMware Workstation

Active Directory (AD) is a key part of many IT infrastructures, making it a great focus for learning and testing vulnerabilities. By setting up Windows Server 2022 as a Domain Controller, you can create a safe lab environment to practice and explore AD security concepts.

This guide will show you how to install and configure Windows Server 2022 for AD vulnerability testing, step by step. Let’s dive in!

---
### Prerequisites

1. **VMware Workstation Pro/Player**: Ensure you have the latest version installed.
2. **Windows Server 2022 ISO file**: Download the ISO file from the [official Microsoft website](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022).
3. **System Resources**: Verify your system meets the minimum requirements:
    - **RAM**: Minimum 4 GB (8 GB or more recommended).
    - **Disk Space**: At least 20 GB of free space for the virtual machine.

---

### Step 1: Create a New Virtual Machine

1. Open **VMware Workstation**.
2. Click on **File** > **New Virtual Machine** or select **Create a New Virtual Machine** from the home screen.
3. Choose **Typical (recommended)** and click **Next**.
![Desktop View](/windows_server(2).webp)
### Step 2: Select the Installer Disc Image

1. Select **Installer disc image file (ISO)**.
2. Browse to the location of your Windows Server 2022 ISO file and click **Next**.
![Desktop View]/(windows_server(3).webp)
### Step 3: Name the Virtual Machine and Set the Location

1. Provide a name for your virtual machine (e.g., "Windows Server 2022").
2. Specify the location where the virtual machine files will be saved.
3. Click **Next**.
![Desktop View](/windows_server(4).webp)
### Step 4: Allocate Disk Space

1. Specify the maximum disk size (e.g., 15 GB or more based on your needs).
2. Click **Next**.
![Desktop View](/windows_server(5).webp)
### Step 5: Customize Hardware (Optional)

1. Uncheck **Power on this virtual Machine** and click **Finish**.
![Desktop View](/windows_server(6).webp)
2. Remove the Floppy from the **Virtual Machine Settings**:
![Desktop View](/windows_server(7).webp)
Click **Ok** after making adjustments.

---

### Step 6: Start the Virtual Machine

1. Select your newly created virtual machine in VMware Workstation.
2. Click **Power on the virtual machine** and press **Enter** when the **Press any key to boot from CD or DVD** message appears.

![Desktop View](/windows_server(8).webp)

---

### Step 7: Install Windows Server 2022

1. Once the virtual machine powers on, the Windows Server 2022 setup will begin.
2. Choose your preferred language, time format, and keyboard input method, then click **Next**.
3. Click **Install Now**.
![Desktop View](/windows_server(12).webp)
2. Choose the virtual disk you created earlier and click **Next**.
![Desktop View](/windows_server(13).webp)
The installation process will start, and the virtual machine will restart several times.

---

### Step 9: Finalize Setup

1. After installation, configure the following:
    - Create an administrator password.
    - Log in with the administrator account.
![Desktop View](/windows_server(14).webp)
---

### Step 10: Post-Installation Configuration

1. **Install VMware Tools**:
    - Go to the **VM** menu and select **Install VMware Tools**.
![Desktop View](/windows_server(17).webp)
    - Double click the VMware tools Follow the on-screen instructions to enhance performance and enable features like shared clipboard and drag-and-drop functionality.
![Desktop View](/windows_server(18).webp)

2. **Optional: Configure Network Settings**:
    - Set a static IP address or join the server to your domain as required. 


Your Windows Server 2022 virtual machine is now fully set up and ready for use.