# Home Backups
This repository contains the backbone of my backup strategy.

I don't own a NAS (although i'm planning on building one soon), so in the meantime, I roughly follow the [3-2-1 backup strategy](https://www.backblaze.com/blog/the-3-2-1-backup-strategy/).

To accomplish this, I run a primary and secondary backup server on Raspberry Pis. With the price of NAND in a freefall, I considered setting them up in software RAID 1, but for simplicity, they are both in a non-RAID configuration.

# How it works
For each device I wish to back up with this method, I add the backup server's public key to the `/root/.ssh/authorized_keys` file on the target machine. 

Via a cronjob, the `backup-entire-remote-machine` script is run and syncs the remote file systems from my target machines to the backup device via rsync. After all my target machines have been sync'd, the primary backup server replicates to the secondary.

Weekly, the primary server pushes changes to my data to Backblaze B2. The cloud sync uses rclone under the hood, so the provider can be any of which rclone supports, however after testing several (AWS, Storj, Backblaze), I eventually settled on Backblaze as there are no costs to upload and storage costs in even their hot tiers are comparable to cold tiers elsewhere. Just don't plan on tons of egress from the cloud backup, it can get pricey!

# Security Model
For remote access, i'm being extra cautious. These machines hold keys for most of my devices. Because of that, I've employed a dual two factor authentication model. In order to SSH into a backup device, I need all of the following:
- Something you know: the user password
- Something you have: the public key for the backup device AND my 2FA enrolled moble device. I use the [Duo pam_duo unix module](https://duo.com/docs/duounix) to enable Duo for SSH.

Physical access is an option if remote access becomes an issue for any reason such as:
- The RPi boot filesystem on the SD card becomes corrupt (which happens too much)
- The RPi itself becomes degraded or out-of-service
- I forget the password or lose the public key

The best option to restore access to my data at that point would be to either transfer the hard drive to another machine or reflash the SD with the contents of this repository and re-enroll the device as a backup device.
