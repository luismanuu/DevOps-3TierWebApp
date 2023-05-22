# DevOps-3TierWebApp
3 Tier Web Application DevOpsPBL

## Step 1 — Prepare NFS Server

1. Launch an EC2 instance as the "Web Server" and create 3 volumes of 10 GiB each in the same AZ.
Check https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html for more info.
![image](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/8db7dd53-a0fa-4c18-8dfd-772a5f79c2a0)
2. Attach all three volumes to the Web Server EC2 instance.
![image](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/267076db-b5f0-42d4-b985-85992f8519dc)


3. Use `lsblk` command to inspect the attached block devices (likely named xvdf, xvdh, xvdg).
![image](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/b577390d-35f0-4a3e-9242-d4b2bdea6766)


4. Create a single partition on each disk using `gdisk` utility. 
`sudo gdisk /dev/xvdf`
![Screenshot 2023-05-21 210851](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/6b14cecc-5f84-464f-9e63-14a5622d945a)




5. Configure the partitions on all three disks and exit the `gdisk` console.

6. Install `lvm2` package using `sudo yum install lvm2` and use `sudo lvmdiskscan` to check available partitions.

7. Mark each disk as a physical volume (PV) using `pvcreate` utility:
   - `sudo pvcreate /dev/xvdf1`
   - `sudo pvcreate /dev/xvdg1`
   - `sudo pvcreate /dev/xvdh1`

8. Verify the creation of the physical volumes using `sudo pvs`.

9. Create a volume group (VG) named `nfs-vg` and add all three PVs using `vgcreate` utility:
   - `sudo vgcreate nfs-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

10. Verify the creation of the volume group using `sudo vgs`.

11. Create three logical volumes (LVs) using `lvcreate` utility:
    - `sudo lvcreate -n lv-opt -L 4G nfs-vg`
    - `sudo lvcreate -n lv-apps -L 4G nfs-vg`
	- `sudo lvcreate -n lv-logs -L 4G nfs-vg`

12. Verify the creation of the logical volumes using `sudo lvs`.

13. Verify the entire setup by running `sudo vgdisplay -v` and `sudo lsblk`.

14. Format the logical volumes with xfs filesystem :
    - `sudo mkfs -t xfs /dev/nfs-vg/lv-opt`
    - `sudo mkfs -t xfs /dev/nfs-vg/lv-apps`
	 - `sudo mkfs -t xfs /dev/nfs-vg/lv-logs`

15. Create  directories to store  files: 
`sudo mkdir -p /mnt/apps`
`sudo mkdir -p /mnt/logs`
`sudo mkdir -p /mnt/opt`

16. Create `/home/recovery/logs` directory to store log backups: `sudo mkdir -p /home/recovery/logs`

17. Mount `/mnt/apps` on `lv-apps` logical volume: `sudo mount /dev/nfs-vg/lv-apps /mnt/apps`

18. Backup log files from `/mnt/logs` to `/home/recovery/logs` using `rsync` utility:
    `sudo rsync -av /mnt/logs/. /home/recovery/logs/`

19. Mount `/mnt/log` on `lv-logs` logical volume: `sudo mount /dev/nfs-vg/lv-logs /mnt/logs`

20. Restore log files from the backup directory: `sudo rsync -av /home/recovery/logs/. /mnt/log`

21. Mount `/mnt/opt` on `lv-opt` logical volume: `sudo mount /dev/nfs-vg/lv-opt /mnt/opt`

22. Update `/etc/fstab` file to persist the mount configuration after server restart.


23. Obtain the UUID of the device using `sudo blkid`.
 ![Screenshot 2023-05-21 213324](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/c1c3941a-6352-4a23-9abe-9c06c1e936f4)


24. Open the /etc/fstab file using `sudo vi /etc/fstab`.

25. Update the /etc/fstab file with the UUID of the device, removing the leading and ending quotes.
![Screenshot 2023-05-21 213336](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/7a85b439-b3ce-4ada-bd96-20383530e568)

26. Test the configuration and reload the daemon:
   - `sudo mount -a`
   - `sudo systemctl daemon-reload`

27.Verify your setup by running df -h, output must look like this
![Screenshot 2023-05-21 213629](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/75db4317-3221-4470-9f4b-b60faae21a22)
