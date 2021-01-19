# postmarketOS-netboot-notes
#### Netbooting postmarketOS

pc:
sudo ./startloader.sh
(
sudo ./fusee-launcher/fusee-launcher.py ./payload/uart_payload_n7.bin -P 7330
sudo ./utils/nvflash_v1.13.87205_miniloader_patched --setbct --bct ./bct/nexus_7_grouper_bct.bin --configfile ./utils/flash.cfg --bl ./bootloader/bootloader-grouper-4.23.img --go
sudo ./utils/nvflash_v1.13.87205_miniloader_patched --resume --download LNX flatline_grouper.img --configfile ./utils/flash.cfg
)
sudo fastboot boot nexusblobs/boot.img
telnet 172.16.42.1
grouper:
nc -l -p 1234 > alpine.tar.gz

pc:
nc -w 3 172.16.42.1 1234 < alpine-armhf-chroot.tar.gz

grouper:
tar -xvf alpine.tar.gz
chroot alpine
route add default gw 172.16.42.2
echo nameserver 8.8.8.8 | tee /etc/resolv.conf

pc:
sysctl net.ipv4.ip_forward=1
iptables -P FORWARD ACCEPT
iptables -A POSTROUTING -t nat -j MASQUERADE -s 172.16.42.0/24

grouper:
date -s *date*
apk upgrade
apk add nbd-client

pc:
nbd-server 172.16.42.2:9998 /home/nergzd/tegra30_debrick/nexusblobs/pmOS-rootfs.img

grouper:
nbd-client 172.16.42.2 9998 /dev/nbd0
exit
mount_boot_partition
Ctrl^C
mount /dev/nbd0p1 /boot
extract_initramfs_extra boot/initramfs-asus-grouper-extra
pmos_continue_boot
