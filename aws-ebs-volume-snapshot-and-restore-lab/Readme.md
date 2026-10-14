# AWS EBS Volume, Snapshot, and Restore Lab

## Project Overview

This lab demonstrates how to manage Amazon Elastic Block Store (Amazon EBS) volumes attached to an Amazon EC2 Linux instance. It covers creating and attaching a storage volume, configuring a Linux file system, mounting persistent storage, creating an EBS snapshot, and restoring deleted data from the snapshot.

The goal is to understand how AWS block storage works and how EBS snapshots can support backup, recovery, and data protection strategies.

## Objectives

- Create an Amazon EBS volume.
- Attach an EBS volume to an EC2 instance.
- Connect to a Linux instance using EC2 Instance Connect.
- Create and mount an ext3 file system.
- Configure persistent mounting through `/etc/fstab`.
- Create an EBS snapshot.
- Restore an EBS volume from a snapshot.
- Verify data recovery after deleting a file from the original volume.

## AWS Services and Technologies

- **Amazon EC2:** Virtual server used to host the Linux environment.
- **Amazon EBS:** Persistent block storage attached to the EC2 instance.
- **EBS Snapshots:** Point-in-time backups used to restore volume data.
- **EC2 Instance Connect:** Browser-based connection to the Linux instance.
- **Linux:** File system management, mounting, and storage verification.
- **ext3:** Linux journaling file system used in this lab.

## Architecture and Workflow

1. Create a new 1 GiB EBS volume.
2. Attach the volume to the Lab EC2 instance.
3. Connect to the instance using EC2 Instance Connect.
4. Format the volume with an ext3 file system.
5. Mount the volume at `/mnt/data-store`.
6. Configure `/etc/fstab` for persistent mounting.
7. Create a test file on the mounted volume.
8. Create an EBS snapshot.
9. Delete the original test file.
10. Create a new EBS volume from the snapshot.
11. Attach and mount the restored volume.
12. Verify that the deleted file is available in the restored volume.

## Step-by-Step Implementation

### Task 1: Create an EBS Volume

1. Open the **Amazon EC2 Console**.
2. Navigate to **Elastic Block Store → Volumes**.
3. Choose **Create volume**.
4. Configure the following settings:

| Setting | Value |
|---|---|
| Volume type | General Purpose SSD (gp2) |
| Size | 1 GiB |
| Availability Zone | Same AZ as the Lab EC2 instance |
| Name tag | `My Volume` |

5. Choose **Create volume**.
6. Wait until the volume state becomes `Available`.

**Key concept:** An EBS volume must be in the same Availability Zone as the EC2 instance to which it is attached.

### Task 2: Attach the Volume

1. Select `My Volume`.
2. Choose **Actions → Attach volume**.
3. Select the `Lab` EC2 instance.
4. Enter `/dev/sdb` as the device name.
5. Choose **Attach volume**.

Verify that the volume state changes to `In-use`.

> Note: On Nitro-based EC2 instances, the operating system may expose an attached EBS volume using an NVMe device name such as `/dev/nvme1n1`, rather than the requested EC2 device name `/dev/sdb`.

### Task 3: Connect to the EC2 Instance

1. Navigate to **EC2 → Instances**.
2. Select the `Lab` instance.
3. Choose **Connect**.
4. Open the **EC2 Instance Connect** tab.
5. Choose **Connect** to open a terminal.

Check the available storage:

```bash
df -h
```

The output should show the existing root volume. The new EBS volume will not appear as a mounted file system yet.

### Task 4: Create and Mount the File System

**1. Identify the attached volume**

Before formatting, identify the correct device:

```bash
lsblk
```

Confirm which device corresponds to the newly attached 1 GiB volume.

**2. Create an ext3 file system**

For an environment where the new volume is confirmed to be `/dev/sdb`, run:

```bash
sudo mkfs -t ext3 /dev/sdb
```

**Warning:** Formatting a device destroys existing file system data. Verify the device before running this command.

**3. Create the mount directory**

```bash
sudo mkdir -p /mnt/data-store
```

**4. Mount the volume**

```bash
sudo mount /dev/sdb /mnt/data-store
```

**5. Configure persistent mounting**

Add the following entry to `/etc/fstab`:

```bash
echo "/dev/sdb /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab
```

Verify the configuration:

```bash
cat /etc/fstab
```

Check the mounted storage:

```bash
df -h
```

**Important:** Device names can change or differ between EC2 instance types. For production systems, consider using the file system UUID in `/etc/fstab` instead of relying on `/dev/sdb`. Verify the configuration before rebooting.

**6. Create a test file**

```bash
sudo sh -c "echo 'Some text has been written' > /mnt/data-store/file.txt"
```

Read the file:

```bash
cat /mnt/data-store/file.txt
```

Expected output:

```text
Some text has been written
```

### Task 5: Create an EBS Snapshot

1. Return to **EC2 → Volumes**.
2. Select `My Volume`.
3. Choose **Actions → Create snapshot**.
4. Add the following tag:

| Key | Value |
|---|---|
| Name | `My Snapshot` |

5. Choose **Create snapshot**.
6. Navigate to **Snapshots** and wait for the snapshot status to become `Completed`.

An EBS snapshot provides a point-in-time backup of the volume's data.

**Delete the test file from the original volume:**

```bash
sudo rm /mnt/data-store/file.txt
```

Verify that the file no longer exists:

```bash
ls /mnt/data-store/file.txt
```

The command should report that the file does not exist.

### Task 6: Restore the Volume from the Snapshot

#### Task 6.1: Create a Volume from the Snapshot

1. Navigate to **EC2 → Snapshots**.
2. Select `My Snapshot`.
3. Choose **Actions → Create volume from snapshot**.
4. Select the same Availability Zone as the Lab instance.
5. Add the following tag:

| Key | Value |
|---|---|
| Name | `Restored Volume` |

6. Choose **Create volume**.
7. Wait for the new volume to become `Available`.

#### Task 6.2: Attach the Restored Volume

1. Navigate to **Volumes**.
2. Select `Restored Volume`.
3. Choose **Actions → Attach volume**.
4. Select the `Lab` EC2 instance.
5. Specify `/dev/sdc` as the device name.
6. Choose **Attach volume**.

Wait for the volume state to become `In-use`.

#### Task 6.3: Mount and Verify the Restored Volume

Create a mount directory:

```bash
sudo mkdir -p /mnt/data-store2
```

Identify the actual device name using:

```bash
lsblk
```

Mount the restored volume. In the lab environment, use `/dev/sdc` if that is the correct device:

```bash
sudo mount /dev/sdc /mnt/data-store2
```

Check whether the original file has been recovered:

```bash
ls -l /mnt/data-store2/file.txt
```

Read the recovered file:

```bash
cat /mnt/data-store2/file.txt
```

Expected output:

```text
Some text has been written
```

The file is available because the snapshot preserved the data from before the file was deleted.

## Key Learnings

- EBS provides persistent block storage for EC2 instances.
- EBS volumes are Availability Zone-specific.
- A volume must be formatted before it can be used as a new file system.
- Mount points provide access to file systems within Linux.
- The `/etc/fstab` file can configure file systems to mount automatically.
- EBS snapshots support backup, cloning, and recovery.
- Creating a new volume from a snapshot allows recovery without modifying the original volume.
- Device names presented to Linux may differ from the names configured in the EC2 console.




