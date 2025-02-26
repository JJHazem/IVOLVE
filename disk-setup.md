# 🛠️ Disk Management on Ubuntu VM: Partitioning, Filesystem, Swap, and LVM

**Author:** Ahmed Hazem

## 📌 **Objective:**
Attach a 15GB disk to your VM, partition it into 5GB, 5GB, 3GB, and 2GB sections. Use the first 5GB partition as a file system, configure the 2GB partition as swap, initialize the second 5GB as a Volume Group (VG) with a Logical Volume (LV), then extend the LV by adding the 3GB partition.

---

## 🚀 **Step 1: Attach the Disk to Your VM**
Attach a 15GB disk to your VM through your virtualization platform (like VirtualBox, VMware, or a cloud provider). Verify the disk with:

```bash
lsblk
```

You should see a new disk, likely `/dev/sdb`.

---

## 🔧 **Step 2: Partition the Disk**

```bash
sudo fdisk /dev/sdb
```

Follow the prompts to create four partitions:
- `/dev/sdb1`: **5GB** (for filesystem)
- `/dev/sdb2`: **5GB** (for LVM)
- `/dev/sdb3`: **3GB** (for LVM extension)
- `/dev/sdb4`: **2GB** (for swap)

Write the changes and check:

```bash
lsblk
```

---

## 🟩 **Step 3: Create the Filesystem**

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkdir /mnt/data1
sudo mount /dev/sdb1 /mnt/data1
```

Update `/etc/fstab`:

```bash
echo "/dev/sdb1 /mnt/data1 ext4 defaults 0 0" | sudo tee -a /etc/fstab
```

---

## 🟨 **Step 4: Configure Swap Space**

```bash
sudo mkswap /dev/sdb4
sudo swapon /dev/sdb4
```

Add to `/etc/fstab`:

```bash
echo "/dev/sdb4 none swap sw 0 0" | sudo tee -a /etc/fstab
```

Verify:

```bash
free -h
```

---

## 🟧 **Step 5: Set Up LVM**

```bash
sudo pvcreate /dev/sdb2
sudo vgcreate my_vg /dev/sdb2
sudo lvcreate -L 4.5G -n my_lv my_vg
sudo mkfs.ext4 /dev/my_vg/my_lv
```

Mount the LV:

```bash
sudo mkdir /mnt/my_lv
sudo mount /dev/my_vg/my_lv /mnt/my_lv
```

Add to `/etc/fstab`:

```bash
echo "/dev/my_vg/my_lv /mnt/my_lv ext4 defaults 0 0" | sudo tee -a /etc/fstab
```

---

## 🟪 **Step 6: Extend the LVM with the 3GB Partition**

```bash
sudo pvcreate /dev/sdb3
sudo vgextend my_vg /dev/sdb3
sudo lvextend -l +100%FREE /dev/my_vg/my_lv
sudo resize2fs /dev/my_vg/my_lv
```

---

## ✅ **Step 7: Verify the Setup**

```bash
df -h
swapon --show
lvdisplay
lsblk
```

---

## 🎯 **Results:**
- **5GB filesystem mounted at `/mnt/data1`**
- **2GB swap partition active**
- **8GB logical volume (`5GB + 3GB`) mounted at `/mnt/my_lv`**

**Author:** Ahmed Hazem

