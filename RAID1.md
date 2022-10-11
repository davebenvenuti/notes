# Setting up a RAID1

## Use case: one drive at a time

These are instructions for creating an encrypted RAID1 array with a single drive and adding a second drive later on, maybe after copying data over from it, for example.

### Initial 1-drive RAID

1. Wipe the partition table on the drive

```bash
sudo parted /dev/sdx mklabel gpt
```

2. Create a new partition

```bash
sudo fdisk /dev/sdx
```
`n` will create a new partition.  The defaults are fine.
`t 1` to change the type of partition 1.  Pick **Linux RAID**.
`w` to write the new partition table

3. Create the new RAID device

```bash
sudo mdadm --create /dev/md0 --level=mirror --force --raid-devices=1 /dev/sdx1
```

4. Create the LUKS device on the RAID
  
```bash
sudo cryptsetup -y -v luksFormat /dev/md0
```

5.  Open the LUKS device and create a mapping

This will create `/dev/mapper/nameofyourraid`

```bash
sudo cryptsetup luksOpen /dev/md0 nameofyourraid
```

6.  Format the open LUKS format however you'd like

eg:

```bash
sudo mkfs.ext4 /dev/mapper/nameofyourraid
```

7.  Create a mountpoint

```bash
sudo mkdir /media/nameofyourraid
```

8.  Mount

```bash
sudo mount /dev/mapper/nameofyourraid /media/nameofyourraid
```

Done!

### Setup second drive

Steps 1 and 2 will look very familiar.

1. Wipe the partition table on the drive

```bash
sudo parted /dev/sdy mklabel gpt
```

2. Create a new partition

```bash
sudo fdisk /dev/sdy
```
`n` will create a new partition.  The defaults are fine.
`t 1` to change the type of partition 1.  Pick **Linux RAID**.
`w` to write the new partition table

3. Add the new drive as a spare device to the RAID array

```bash
sudo mdadm --add /dev/md0 /dev/sdy1
```

4. Increase the size of the RAID array and balance with the former spare

```bash
sudo madam --grow /dev/md0 --raid-devices=2
```

