# Day 13 – Linux Volume Management (LVM)

### Task : Learn LVM to manage storage flexibly – create, extend, and mount volumes.


1) Go to Instances , open left side pannel --> Volume 
2) Create as many new volume as you want of any size you want 
3) When it is available , select it and attach it to the linux server you're working on   
4) Now come to terminal and check weather the new volume is attached or not ; Run `lsblk`

You'll see 
``` bash
nvme0n1                259:0    0   25G  0 disk
├─nvme0n1p1            259:1    0   24G  0 part /
├─nvme0n1p14           259:2    0    4M  0 part
├─nvme0n1p15           259:3    0  106M  0 part /boot/efi
└─nvme0n1p16           259:4    0  913M  0 part /boot
nvme1n1                259:5    0   10G  0 disk
nvme2n1                259:6    0   12G  0 disk
nvme3n1                259:7    0    5G  0 disk
```

5) New volumes have been attached successfully
6) Now run `lvm` an interactive LVM shell will gets opened , you can run lvm commands here
7) Physical Volume (PV) Commands to run: 
    - `pvcreate /dev/nvme1n1` or `pvcreate physical_vol` 
    - `pvdisplay` : Show PV details 
    - `pvs` : Brief PV info
    - `pvremove /dev/nvme1n1` : remove PV
    - `pvmove` : move data between PVs ( `pvmove /dev/nvme1n1 /dev/nvme2n1` move data from nvme1n1 --> nvme2n1 ) 
8) Volume Group (VG) Commands to run :
    - `vgcreate VGname /dev/nvme1n1 /dev/nvme2n1` : Create a Volume group using nvme1n1 and nvme2n1 physical volumes
    - `vgdisplay` : Show VG details
    - `vgs` : Brief VG info
    - `vgextend VGname /dev/nvme3n1` : Add new PV to VG (New physical volume (nvme3n1) will add to VG)
    - `vgreduce VGname /dev/nvme2n1` : Remove PV from VG (Remove nvme2n1 from VG)
    - `vgrename VGname NewVGname` : Rename a VG
    - `vgremove VGname` : Delete a VG   | Also  `vgremove --force VGname`
    - `vgsplit myvg newvg /dev/nvme1n1` : First attach `/dev/nvme1n1` to `myvg` then perform this split (It'll only take /dev/nvme1n1 and attach it to newvg)
    
9) Logical Volume (LV) Commands to run : 
    - `lvcreate -L 10G -n mylv myvg` : Create a LV named mylv from VG named myvg (If present)
    - `lvdisplay` : Show LV details   
    - `lvs` : Brief LV info   
    - `lvextend -L +5G(Size) /dev/myvg/mylv(LVname)`  : Increase the size of LV by 5gb
    - `lvextend -L 20G /dev/myvg/mylv` : Set total to 20 GB  
    - `lvreduce -L -5G /dev/myvg/mylv` : Remove/Decrease 5 GB
    - *Note* : Whenever use lvextend always put -r flag to Resize filesystem too (df -h)
    - `lvresize -r -L +5G /dev/myvg/mylv` : With filesystem resize , as I said in *Note*
    - `lvrename myvg mylv newlv` or `lvrename /dev/myvg/mylv /dev/myvg/newlv`: Renaming a LV 
    - `lvremove /dev/myvg/mylv` : Delete a LV
    - If you extend you LV by using lvextend but it is not showing in df -h , then run `resize2fs /dev/myvg/mylv` It will update the filesystem (df -h)

10) Mount the Logical Volume 
    - `mkdir /data` : Make a folder where you want to use it as endpoint
    - `mount /dev/datavg/datalv /data` : Mount the LV to that folder 

11) Unmount the Logical Volume 
    - `umount /dev/myvg/mylv` : The storage will unmount from the end point

12) You can Mount and Unmount the LV whenever you want the data won't get lost (It is like a external device : Like a pendrive)

13) Now /data is the drive/storage/Lv whatever you say --> It is the extra storage you want 
    
14) By using these commands you can Manage LVM and there you go... Now you can use your extra storage/space    





