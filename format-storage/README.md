#Format storage device(linux)
---
The process involves three main steps:
1.  **Identify** the correct device name for your pendrive.
2.  **Partition** the drive (create a container for the filesystem).
3.  **Format** the partition with a filesystem and give it a name (label).

---

### Step 1: Identify Your Pendrive

This is the most critical step to avoid formatting the wrong drive.

1.  Open a terminal.
2.  Run the `lsblk` command **before** plugging in your pendrive. You will see your internal drives (like `nvme0n1` or `sda`).
3.  Now, **plug in your pendrive**.
4.  Run `lsblk` again. You will see a new device appear. This is your pendrive. It will likely be named `/dev/sdb`, `/dev/sdc`, etc.

**Example `lsblk` output:**

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
...
sdb           8:16   1  28.7G  0 disk  <-- This is the pendrive device
└─sdb1        8:17   1  28.7G  0 part /run/media/user/OLD_NAME
nvme0n1     259:0    0 931.5G  0 disk
├─nvme0n1p1 259:1    0   512M  0 part /boot
└─nvme0n1p2 259:2    0   931G  0 part /
```

In this example, the pendrive is `/dev/sdb`. The existing partition on it is `/dev/sdb1`.

---

### Step 2: Unmount the Pendrive

If your system automatically mounted the pendrive, you must unmount it before you can format it.

Use the device name you found in Step 1.

```bash
# Unmount the partition, e.g., /dev/sdb1
sudo umount /dev/sdb1

# If there are multiple partitions, unmount them all
sudo umount /dev/sdb*
```
If you get an error "not mounted", that's fine. It just means it wasn't mounted, and you can proceed.

---

### Step 3: Partition the Drive with `fdisk`

We will create a new, clean partition table and one partition that uses the entire drive. We'll use the powerful `fdisk` utility.

1.  Start `fdisk`, targeting the **drive** (e.g., `/dev/sdb`, **NOT** `/dev/sdb1`).
    ```bash
    sudo fdisk /dev/sdb
    ```

2.  You are now inside the `fdisk` interactive prompt. Type the following commands one by one and press Enter.

    *   `g` - This creates a new, empty GPT partition table. (GPT is the modern standard).
    *   `n` - This creates a new partition. `fdisk` will ask you for the partition number, first sector, and last sector. You can just press **Enter** at each prompt to accept the defaults, which will create one partition using the entire drive.
    *   `w` - This **writes** the changes to the drive and exits `fdisk`. This is the point of no return.

The partition `/dev/sdb1` has now been created.

---

### Step 4: Format the Partition and Give it a Name

This is where you create the filesystem and assign the name (a "label"). You need to choose which filesystem to use.

#### Choose Your Filesystem:

*   **exFAT**: The best choice for modern universal compatibility. Works on Windows, macOS, and Linux. No 4GB file size limit. **(Recommended for general use)**.
*   **FAT32 (VFAT)**: The most compatible. Works on virtually everything (TVs, cars, etc.). Has a **4GB maximum file size limit**.
*   **NTFS**: The native Windows filesystem. Good if the drive will be used almost exclusively with Windows.
*   **ext4**: The native Linux filesystem. Best choice if the drive will only be used with Linux systems.

#### Formatting Commands:

Replace `"MY_USB_DRIVE"` with your desired name and `/dev/sdb1` with the partition you created.

**A) To Format as exFAT (Recommended):**
*   First, make sure you have the tools: `sudo pacman -S exfatprogs`
*   Run the format command:
    ```bash
    sudo mkfs.exfat -n "MY_USB_DRIVE" /dev/sdb1
    ```

**B) To Format as FAT32:**
*   Run the format command:
    ```bash
    sudo mkfs.vfat -F 32 -n "MY_USB_DRIVE" /dev/sdb1
    ```

**C) To Format as NTFS:**
*   First, make sure you have the tools: `sudo pacman -S ntfs-3g`
*   Run the format command (note the `-L` for label):
    ```bash
    sudo mkfs.ntfs -f -L "MY_USB_DRIVE" /dev/sdb1
    ```

**D) To Format as ext4 (Linux Only):**
*   Run the format command (note the `-L` for label):
    ```bash
    sudo mkfs.ext4 -L "MY_USB_DRIVE" /dev/sdb1
    ```

After the command finishes, your drive is formatted and named!

---

### Step 5: Verify the Result

You can check your work with `lsblk -f`.

```bash
lsblk -f
```

You should see your new partition with the correct filesystem type (`FSTYPE`) and the name you assigned (`LABEL`).

**Example output after formatting as exFAT:**
```
NAME        FSTYPE FSVER LABEL          UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
...
sdb
└─sdb1      exfat  1.0   MY_USB_DRIVE   ABCD-1234
nvme0n1
...
```

Your pendrive is now ready. You can unplug it and plug it back in, and your system should recognize it with its new name.