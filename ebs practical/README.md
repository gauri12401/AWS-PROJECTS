# AWS EBS Multi-Partition Setup with Persistent Mount Using UUID & fstab

## 📌 Project Overview

This project demonstrates how to configure an Amazon EBS volume on an AWS EC2 instance by creating multiple partitions, formatting them with the ext4 filesystem, and configuring persistent mounts using UUIDs in `/etc/fstab`.

A 100GB gp3 EBS volume was attached to an Ubuntu EC2 instance and partitioned as follows:

* **50GB Partition** → Mounted at `/logs`
* **20GB Partition** → Mounted at `/backup`

The configuration ensures that both partitions automatically mount after every reboot, making the setup production-ready and reliable.

---

## 🎯 Architecture Overview

```text
AWS Console
     │
     ▼
EC2 Instance (Ubuntu 24.04)
┌─────────────────────────────────────┐
│ Root Volume (20GB gp3)              │
│ Mounted on /                        │
│                                     │
│ EBS Volume (100GB gp3)              │
│                                     │
│ ├── nvme1n1p1 (50GB ext4)           │
│ │      └── /logs                    │
│ │                                   │
│ └── nvme1n1p2 (20GB ext4)           │
│        └── /backup                  │
│                                     │
│ UUID-Based Mounting                 │
│ /etc/fstab Configuration            │
└─────────────────────────────────────┘
```

---

## 📊 Storage Configuration

| Component         | Size | Filesystem | Mount Point                    |
| ----------------- | ---- | ---------- | ------------------------------ |
| Root Volume       | 20GB | ext4       | /                              |
| Partition 1       | 50GB | ext4       | /logs                          |
| Partition 2       | 20GB | ext4       | /backup                        |
| Unallocated Space | 30GB | -          | Available for Future Expansion |

---

## ✅ Prerequisites

* AWS Account
* Ubuntu EC2 Instance
* EBS Volume (100GB gp3)
* SSH Access
* Basic Linux Knowledge

---

# 🚀 Implementation Steps

## Step 1: Verify Attached EBS Volume

```bash
lsblk
```

Expected Output:

```text
nvme0n1      20G
└─nvme0n1p1

nvme1n1     100G
```

---

## Step 2: Create Partitions

Open fdisk:

```bash
sudo fdisk /dev/nvme1n1
```

### Create First Partition (50GB)

```text
n
p
1
Enter
+50G
```

### Create Second Partition (20GB)

```text
n
p
2
Enter
+20G
```

Save changes:

```text
w
```

Verify:

```bash
lsblk
```

Expected:

```text
nvme1n1
├─nvme1n1p1   50G
└─nvme1n1p2   20G
```

---

## Step 3: Format Partitions

Format both partitions using ext4:

```bash
sudo mkfs.ext4 /dev/nvme1n1p1
sudo mkfs.ext4 /dev/nvme1n1p2
```

---

## Step 4: Create Mount Points

```bash
sudo mkdir /logs
sudo mkdir /backup
```

---

## Step 5: Mount Partitions

```bash
sudo mount /dev/nvme1n1p1 /logs
sudo mount /dev/nvme1n1p2 /backup
```

Verify:

```bash
df -hT
```

Expected:

```text
/dev/nvme1n1p1   ext4   49G   47G   /logs
/dev/nvme1n1p2   ext4   20G   19G   /backup
```

---

## Step 6: Retrieve UUIDs

```bash
sudo blkid
```

Example:

```text
/dev/nvme1n1p1: UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
/dev/nvme1n1p2: UUID="yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"
```

---

## Step 7: Configure Persistent Mounts

Backup existing fstab:

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

Edit fstab:

```bash
sudo vim /etc/fstab
```

Add the following entries:

```bash
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /logs ext4 defaults,nofail 0 2
UUID=yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy /backup ext4 defaults,nofail 0 2
```

Save and exit.

---

## Step 8: Validate fstab Configuration

```bash
sudo mount -av
```

Output:

```text
/                 : ignored
/boot             : already mounted
/boot/efi         : already mounted
/logs             : already mounted
/backup           : already mounted
```

---

# ✅ Final Verification

Verify mounted filesystems:

```bash
df -hT
```

Sample Output:

```text
Filesystem      Type  Size Used Avail Use% Mounted on
/dev/root       ext4   19G 2.1G   17G  12% /
/dev/nvme1n1p1  ext4   49G 2.1M   47G   1% /logs
/dev/nvme1n1p2  ext4   20G 2.1M   19G   1% /backup
```

Create test files:

```bash
echo "Logs Partition Working" | sudo tee /logs/test.log

echo "Backup Partition Working" | sudo tee /backup/test.txt
```

Verify:

```bash
cat /logs/test.log

cat /backup/test.txt
```

---

## 🔄 Reboot Test

```bash
sudo reboot
```

After reconnecting:

```bash
df -hT
```

Both partitions should automatically mount.

---

## 💡 Production Benefits

| Feature                    | Benefit                                    |
| -------------------------- | ------------------------------------------ |
| Separate Log Storage       | Prevents root volume from filling up       |
| Dedicated Backup Partition | Isolates backup data from application logs |
| UUID-Based Mounting        | Survives reboots and device name changes   |
| gp3 Storage                | Consistent performance with low latency    |
| ext4 Filesystem            | Reliable and widely used Linux filesystem  |
| EBS Snapshots              | Easy backup and disaster recovery          |
| Future Expansion           | 30GB remains available for future growth   |

---

## 🔧 Useful Commands

```bash
# List block devices
lsblk

# Show mounted filesystems
df -hT

# Get UUIDs
sudo blkid

# View fstab
cat /etc/fstab

# Mount all fstab entries
sudo mount -av

# Unmount partitions
sudo umount /logs
sudo umount /backup

# Extend filesystem after resizing
sudo resize2fs /dev/nvme1n1p1
```

---

## 👩‍💻 Author

### Gauri Shambharkar

**AWS | Linux | DevOps | Terraform | Kubernetes**

This project demonstrates AWS EBS storage administration, Linux disk partitioning, filesystem management, UUID-based persistent mounting, and production storage best practices.
