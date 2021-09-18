Simple script to create an xfs volume out of lvg:

```bash
# create vol group lvg1 and add disk vdb to it
sudo vgcreate lvg1 /dev/vdb
# create logical vol data1 using all of lvg1
sudo lvcreate -n data1 -l 100%FREE lvg1
# format data1 and mount to filesystem
sudo mkfs.xfs /dev/lvg1/data1
sudo xfs_admin -L VG1DATA1 /dev/lvg1/data1
sudo mkdir /data1
sudo mount /dev/lvg1/data1 /data1
df -h /data1
# persist mount across reboots
echo 'LABEL=VG1DATA1  /data1  xfs  rw,pquota  0 2' | sed 's/  /\t/g' | sudo tee -a /etc/fstab
```
