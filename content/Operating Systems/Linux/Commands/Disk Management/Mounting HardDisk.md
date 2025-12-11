Got it  You want to mount your HDD drive on Linux (Ubuntu, since that’s what you’re using). I’ll walk you through the steps:

```
sudo mount -t ntfs-3g /dev/sda1 /mnt/HardDrive
sudo ntfsfix /dev/sda1

sudo umount /mnt/HardDrive
sudo udisksctl power-off -b /dev/sda1
```

---

### 1. **Find your drive**

```bash
lsblk -f
```

This will list all block devices. Look for your HDD (usually something like `/dev/sdb`, `/dev/sdc`, etc.).  
Example output:

```
sda      ext4   root
sdb1     ntfs   Data
```

- `sda` → your main OS drive.
- `sdb1` → your HDD partition.
---

### 2. **Create a mount point**
Make a folder where you want the drive to be accessible:
```bash
sudo mkdir /mnt/hdd
```

---

### 3. **Mount the drive manually**

If it’s NTFS:
```bash
sudo mount -t ntfs-3g /dev/sdb1 /mnt/hdd
```

If it’s EXT4:
```bash
sudo mount -t ext4 /dev/sdb1 /mnt/hdd
```

Now you can access your files at `/mnt/hdd`.

---

### 4. **Unmount when done**

```bash
sudo umount /mnt/hdd
```

---

### 5. **Mount automatically at boot (optional)**

If you want it to auto-mount:

1. Get UUID of the drive:
```bash
blkid
```
Example:
```
/dev/sdb1: UUID="A1B2-C3D4" TYPE="ntfs"    
```
2. Edit `fstab`:
```bash
sudo nano /etc/fstab
```
Add a line like:
```
UUID=A1B2-C3D4  /mnt/hdd  ntfs-3g  defaults  0  0
```   
(replace with your UUID and filesystem type).
3. Test:

```bash
sudo mount -a
```
If no errors → it will mount at every boot.   

---
# If errors occure
```bash
sudo ntfsfix /dev/sda1
```