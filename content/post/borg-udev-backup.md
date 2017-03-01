+++

date = "2017-03-01T00:00:00+00:00"
description = "An automated, local BorgBackup setup using udev rules."
title = "Local BorgBackup automation with udev"

+++

**TL;DR**:
An automated, local BorgBackup setup using `udev` rules.
Backup is triggered by the insertion of a specific external hard drive and the insertion of a specific USB flash drive containing the encryption key.

<!--more-->

I wasn't used to doing regular backups because I have never bothered to setup an automated system and the manual process was too tedious to be done regularly.
Sure enough, I have lost data on a few occasions -- luckily never something of great importance -- so I've decided to create a friendly and automated system to prevent a grim scenario.

## Choice

First thing to do was to select the core of my system.
`cp`? `rsync`? RAID? Filesystem snapshots?

After reading a few threads on [/r/Backup](https://www.reddit.com/r/Backup/) my choice came down to [BorgBackup](https://github.com/borgbackup/borg) for several reasons, which are well outlined in its `README`.
Namely, BorgBackup deduplicates, encrypts, compresses, is fast, is userspace, is open source, is maintained and supports remote backups for a future setup.
It is exactly what I was looking for, in a well documented, user-friendly package.

## Use case

Since I'm not a fan of backing up to the cloud, unless I own that cloud, and since I don't own a remote machine with enough storage space, local backups are the way to go for me.
Furthermore, my primary computer is not a desktop, but a laptop, so having a disk for backups constantly connected is impractical.
Therefore, whenever I can and remember, I connect an external drive over USB to backup to.

The most secure encryption option that BorgBackup provides is to use a 256-bit AES encryption key and a password which unlocks that AES key.
I would have loved to have an automated setup which doesn't require user intervention, apart from connecting the external drive, but keeping both the password and the encryption key on my laptop seems insecure.
To get around that issue I have separated the password from the key.
Password is kept on my laptop, but the key is kept on a USB flash drive, which needs to be plugged into the laptop at the same time as the external disk for backups to happen.
This definitely isn't the most secure setup, as decrypting the backup doesn't require something I know and something I have, but two items that I have, however it provides me with what I want -- an automated backup with no user intervention -- without being as insecure as storing both the password and the key on my laptop.

To recap, the backup should start whenever I insert both the external drive and the USB flash drive into my laptop and shouldn't require any other user intervention.

## Backup script

It made sense to start with a Bash script that calls `borg` and makes a backup, because I used to run the backup manually initially.

First, I started off with defining a few variables for ease of use.
My OS is Ubuntu, which automatically mounts newly inserted media to `/media/user/uuid-of-the-partition/`.
UUID of a partition doesn't change after the partition is formatted, so using UUIDs (or serial numbers in FAT32's case) is a good and constant reference.

    #!/bin/bash
    # Location of the repository. Directory borg in a partition on the external HDD.
    REPOSITORY="/media/luka/75824427-9211-4aca-90e1-f739c91e1afc/borg"
    # Location of the directory containing the AES key. A partition on the USB flash drive.
    KEYDIR="/media/luka/9A88-8443"
    # Password to unlock the AES key.
    PASSPHRASE="YOUR_PASSWORD"
    # The external drive isn't a fast one, thus some compression is useful.
    COMPRESSION="zlib,6"
    # Name of the newly created archive. Must be unique in the repository, hence the placeholder {now} is used.
    ARCHIVE="{now}"
    # What will get backed up.
    TARGETS+="/i/want/to/backup/this "  # Note the space at the end.
    TARGETS+="/and/this "               # Note the space at the end.

After the variables are defined, a call to `borg` needs to be done.

    # Make the variables accessible to the borg executable.
    export BORG_KEYS_DIR=$KEYDIR
    export BORG_PASSPHRASE=$PASSPHRASE
    # Make a new archive.
    borg create --verbose --stats --progress --compression=$COMPRESSION $REPOSITORY::$ARCHIVE $TARGETS
    # Check if everything is ok.
    borg check --verbose --last=1 $REPOSITORY

## `udev` rules

Once I was happy with my simple backup script, I wanted to run it when the two drives are inserted.
I used `udevadm` to get identifiable data about the two partitions in question and finally resorted to using UUIDs.
When a drive is inserted, a lot of events happen as the device tree is constructed.
By using a UUID, it was easy to single out the partition addition event.

The logic of detecting whether both devices are plugged into the laptop had to be moved to the backup script itself (changes detailed later), because `udev` rules and devices don't share any evironment variables.
For easier detection of the drives from Bash, block device symlinks are created.
Those are `/dev/backup-disk` and `/dev/backup-key`, for the backup HDD and the USB flash drive with the key, respectively.

Finally, I would like to have a visual indicator of the backup's status.
The simplest thing that comes to mind is launching a `gnome-terminal` in which the backup script would be run.
[`xpub`](https://github.com/Ventto/xpub), a Shell script that gets user's display environment variables, makes running programs in the context of the current X session quite easy, as shown below.
Without it, everything under `RUN` in the `udev` rules would be executed in the context of `root`.

    # Match the addition of the partition from the external hard disk drive.
    ACTION=="add", ENV{ID_FS_UUID}=="75824427-9211-4aca-90e1-f739c91e1afc", \  # Note the backslash.
    # Create the /dev/backup-disk symlink.
    SYMLINK+="backup-disk", \
    # Import the X environment variables.
    IMPORT{program}="/usr/bin/xpub", \
    # Run the target script as the X user.
    RUN+="/bin/su $env{XUSER} -c '/home/luka/.start-backup-udev.sh'"

    # Same comments as above apply.
    ACTION=="add", ENV{ID_FS_UUID}=="9A88-8443", \
    SYMLINK+="backup-key", IMPORT{program}="/usr/bin/xpub", RUN+="/bin/su $env{XUSER} -c '/home/luka/.start-backup-udev.sh'"

A thing to note at this point is that the `.start-backup-udev.sh` script only launches the `gnome-terminal` which invokes the actual backup script.
Contents of `.start-backup-udev.sh` could have been written in the `udev` rules directly, but a lot less escaping is needed this way and it looks a bit prettier.
The `.start-backup-udev.sh` script looks like this:

    #!/bin/bash
    gnome-terminal -e "bash -c \"/home/luka/.backup.sh && echo Press any key to quit. && read -sn 1;\""

## Addition to the backup script

Due to the way that `udev` works, the backup script needs to check if both drives are present.
The assumption is that if the drives are present, their partitions were automounted in `/media/user` directory, which is Ubuntu's default behavior.
Two new variables define the device names.

    BACKUP_DISK="/dev/backup-disk"
    BACKUP_KEY="/dev/backup-key"

Before invoking `borg`, a check is added.

    # If any of $BACKUP_DISK and $BACKUP_KEY isn't present, exit with code 1.
    if [ ! -b $BACKUP_DISK ] || [ ! -b $BACKUP_KEY ]
    then
      exit 1
    fi

## Complete code and other resources

The Bash scripts and the `udev` rules can be found [in my GitHub repo](https://github.com/lstrz/borg-udev-automation).

A very useful resource when writing `udev` rules is the [Writing `udev` rules](http://www.reactivated.net/writing_udev_rules.html) guide, and, of course, Arch Linux [wiki pages on udev](https://wiki.archlinux.org/index.php/udev).

Remember that the Bash scripts need to be marked executable!
