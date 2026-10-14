# AWS EBS Volume, Snapshot, and Restore Lab

## Project Overview

This lab demonstrates how to create and manage Amazon Elastic Block Store (Amazon EBS) volumes, attach a volume to an Amazon EC2 instance, format and mount the volume, create a snapshot, and restore data by creating a new volume from the snapshot.

Amazon EBS provides persistent block storage for EC2 instances. EBS snapshots help protect data and support backup and recovery operations.

## Objectives

- Create an Amazon EBS volume.
- Attach an EBS volume to an EC2 instance.
- Format the volume with the ext3 file system.
- Mount the volume to a Linux directory.
- Create an EBS snapshot.
- Create a new EBS volume from a snapshot.
- Attach and mount the restored volume.
- Understand basic EBS backup and recovery operations.

## AWS Services and Tools

- **Amazon EC2:** Virtual server used to perform the lab.
- **Amazon EBS:** Persistent block storage for the EC2 instance.
- **EBS Snapshots:** Point-in-time backups of EBS volumes.
- **Linux Commands:** Used to format, mount, and verify storage devices.

## Lab Architecture

The lab follows this workflow:

1. Create an EBS volume named `My Volume`.
2. Attach the volume to an EC2 instance as `/dev/sdb`.
3. Create an ext3 file system.
4. Mount the volume at `/mnt/data-store`.
5. Create a snapshot named `My Snapshot`.
6. Create a new volume named `Restored Volume` from the snapshot.
7. Attach the restored volume as `/dev/sdc`.
8. Mount the restored volume at `/mnt/data-store2`.

> Note: Linux device names can differ from the names configured in the EC2 console. Always verify the actual device using commands such as `lsblk` before formatting or mounting a disk.

## Step 1: Create an EBS Volume

1. Open the AWS Management Console.
2. Navigate to **EC2 → Elastic Block Store → Volumes**.
3. Choose **Create volume**.
4. Select the required volume type and size.
5. Select the same Availability Zone as the target EC2 instance.
6. Add the name tag `My Volume`.
7. Create the volume.

**Key concept:** An EBS volume must be in the same Availability Zone as the EC2 instance to which it is attached.

## Step 2: Attach the EBS Volume

1. Select `My Volume`.
2. Choose **Actions → Attach volume**.
3. Select the target EC2 instance.
4. Specify the device name `/dev/sdb`, if available.
5. Choose **Attach volume**.

Connect to the EC2 instance and inspect the available block devices:

```bash
lsblk
```

Identify the newly attached EBS device before continuing.

## Step 3: Create a File System

Create an ext3 file system on the new, empty volume.

```bash
sudo mkfs -t ext3 /dev/sdb
```

**Important:** Use this command only after verifying the device name and confirming that the volume is empty. Formatting a device destroys existing file system data.

Verify the file system:

```bash
sudo file -s /dev/sdb
```

## Step 4: Mount the Volume

Create the mount directory:

```bash
sudo mkdir -p /mnt/data-store
```

Mount the volume:

```bash
sudo mount /dev/sdb /mnt/data-store
```

Verify the mount:

```bash
df -h
```

You can also inspect mounted file systems with:

```bash
mount | grep data-store
```

## Step 5: Create a Test File

Create a sample file to verify that the volume stores data:

```bash
sudo touch /mnt/data-store/test-file.txt
```

Add some content:

```bash
echo "EBS snapshot and recovery test" | sudo tee /mnt/data-store/test-file.txt
```

Verify the file:

```bash
cat /mnt/data-store/test-file.txt
```

## Step 6: Create an EBS Snapshot

1. Return to **EC2 → Elastic Block Store → Volumes**.
2. Select `My Volume`.
3. Choose **Actions → Create snapshot**.
4. Enter `My Snapshot` as the snapshot name.
5. Create the snapshot.
6. Navigate to **Snapshots** and monitor its progress until it is completed.

**Key concept:** An EBS snapshot is a point-in-time backup of a volume. Snapshots can be used to create new EBS volumes for recovery or duplication.

## Step 7: Create a Volume from the Snapshot

1. Navigate to **EC2 → Elastic Block Store → Snapshots**.
2. Select `My Snapshot`.
3. Choose **Actions → Create volume from snapshot**.
4. Select an Availability Zone compatible with the target EC2 instance.
5. Configure the required volume settings.
6. Create the volume.
7. Add the name tag `Restored Volume`.

Wait until the restored volume is in the **Available** state.

## Step 8: Attach the Restored Volume

1. Navigate to **Volumes**.
2. Select `Restored Volume`.
3. Choose **Actions → Attach volume**.
4. Select the same EC2 instance.
5. Specify `/dev/sdc`, if available.
6. Attach the volume.

On the EC2 instance, inspect the devices:

```bash
lsblk
```

Identify the restored volume. Do not format it: the snapshot already contains the original file system and data.

## Step 9: Mount the Restored Volume

Create a new mount directory:

```bash
sudo mkdir -p /mnt/data-store2
```

Mount the restored volume using its verified Linux device path:

```bash
sudo mount /dev/sdc /mnt/data-store2
```

Check the mounted volume:

```bash
df -h
```

If the device appears under a different name, replace `/dev/sdc` with the correct device path shown by `lsblk`.

## Step 10: Verify the Restored Data

List the restored files:

```bash
ls -la /mnt/data-store2
```

Read the test file:

```bash
cat /mnt/data-store2/test-file.txt
```

Expected output:

```text
EBS snapshot and recovery test
```

If the file is present and its contents match, the snapshot-based restoration was successful.

## Troubleshooting

### Device Not Found

Run:

```bash
lsblk
```

Confirm that the volume is attached and identify the actual Linux device path.

### Mount Fails

Inspect the file system:

```bash
sudo file -s /dev/sdb
```

Confirm that the correct volume is attached and formatted. For the restored volume, do not run `mkfs`; it should already contain the file system from the snapshot.

### Volume and EC2 Availability Zones Do Not Match

EBS volumes can only be attached to EC2 instances in the same Availability Zone. Create or restore the volume in the appropriate Availability Zone.

### Restored File Is Missing

Verify that the snapshot was created from the correct volume and that the restored volume was created from that snapshot. Confirm that you mounted the restored volume rather than the original volume.

## Key Takeaways

- Amazon EBS provides persistent block storage for EC2 instances.
- EBS volumes must be in the same Availability Zone as the EC2 instance to which they are attached.
- Linux file systems must be created before an empty volume can be used.
- Mounting makes a file system accessible through a directory.
- EBS snapshots provide a way to back up volume data.
- A new EBS volume can be created from a snapshot to restore data.
- A restored volume should not be formatted because formatting would erase its existing file system and data.
- Always verify Linux device names before performing storage operations.

## Cleanup

To avoid unnecessary AWS charges after completing the lab:

1. Unmount the volumes if they are no longer needed:

   ```bash
   sudo umount /mnt/data-store2
   sudo umount /mnt/data-store
   ```

2. Detach the EBS volumes from the EC2 instance.
3. Delete the lab volumes if you no longer need them.
4. Delete the snapshot if you no longer need the backup.

**Warning:** Deleting a volume or snapshot permanently removes that resource. Verify that you no longer need the data before deleting it.

## Conclusion

This lab provided hands-on experience with Amazon EBS volume creation, attachment, Linux file system management, mounting, snapshot creation, and recovery. These skills are important for AWS infrastructure administration, data protection, disaster recovery, and preparation for the AWS Certified Solutions Architect – Associate (SAA-C03) exam.


