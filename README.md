# synology-dsm7-volume1-has-become-read-only

This is my step-by-step solution to "Volume 1 has become read-only" on BRTFS volume on Synology DSM7.

# Important Notes: 
* This is a solution that worked FOR ME. I am not taking responsibility for any data loss. `btrfsck` 
* It is a modified manual from https://github.com/jmiller0/how-to-fsck-on-synology-dsm7, but since it covers different filesystem I found it suitable to publish it separately

# Symptoms

* Synology starts beeping 
* `Volume 1 has become read-only` message in notifications in DSM
* `The system failed to convert Volume 1 to read/write mode.` when trying to use **Convert the Volume to Read/Write**

# Assumptions

* You have access to your Synology over ssh and you have user with sudo powers
* Your volume is named Volume_1 - if not, adjust commands accordingly 
* Your volume is using btrfs filesystem - if not, follow jmiller0's manual

Unless stated otherwise, execute provided commands over Synology ssh.

# Solution

#### 1.  Unmount volume and set DSM to skip checking and mounting it on startup
```
sudo synospace --stop-all-spaces
sudo touch /tmp/volume_skip_check
sudo synosetkeyvalue /etc/synoinfo.conf disable_volumes volume1
```

#### 2. Restart DSM7 from web interface and reconnect SSH tunnel

#### 3. Confirm volume path, via `sudo dmsetup table`, its name should be visible when running `ls /dev/mapper`

#### 4. Run btrfsck repair command:

```sudo btrfsck --repair /dev/mapper/vg1-volume_1 -p```

If you encounter an error `couldn't open RDWR because of unsupported option features (3)`, run commands below and retry:

```
sudo btrfsck --clear-space-cache v1 /dev/mapper/vg1-volume_1 -p
sudo btrfsck --clear-space-cache v2 /dev/mapper/vg1-volume_1 -p
```

**NOTE:** `--clear-space-cache` is supposedly deprecated, btrfsck man page suggests using `btrfs rescue clear-space-cache`, however commands above worked for me.

#### 5. If `btrfsck` finishes with no errors, mount repaired volume

```mount /volume1```



