# Managing linux disk volumes

- fidsk -> pv -> vg > lv -> make fs -> mount
```bash
# list disk partitions
$ ls /dev/nvme*
$ fidsk -l

# check volume group
$ vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <950.82 GiB
  PE Size               4.00 MiB
  Total PE              243409
  Alloc PE / Size       51200 / 200.00 GiB
  Free  PE / Size       192209 / <750.82 GiB
  VG UUID               BEOtCd-k1tG-w03M-467u-M2zz-nhve-q4BanG

# check logical volume
$ lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                Vob60r-WOgD-8tPz-m3za-oMb3-uzZR-cnQmRv
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2023-08-18 12:28:52 +0000
  LV Status              available
  # open                 1
  LV Size                200.00 GiB
  Current LE             51200
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

# create logical volume lv2 100G
$ lvcreate -L 100G --name ubuntu-lv2 ubuntu-vg

$ mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv2

# mount lv2 on /data directory
$ mount /dev/mapper/ubuntu--vg-ubuntu--lv2 /data

$ df -h
```
---
- lvextend
```bash
# extend size of lv2 50G
$ lvextend -L+50G /dev/ubuntu-vg/ubuntu-lv2

# apply filesystem
$ resize2fs /dev/ubuntu-vg/ubuntu-lv2
```
- lvcreate
- vgextend
- pvextend
- lvremove
