## Permanent disable fn-mode for Keychron K2 in Ubuntu
1. If this returns "hid_apple", you are using hid_apple ```ls /sys/module | grep hid_apple```
1. ```echo 2 | sudo tee -a /sys/module/hid_apple/parameters/fnmode```
1. Add the option for the fn key ```echo options hid_apple fnmode=2 | sudo tee -a /etc/modprobe.d/hid_apple.conf```
1. Update initramfs bootfile ```sudo update-initramfs -u -k all```
1. Reboot to test (optional)
1. fn + x + l

## Permament disable fn-mode for Logitech K400+ in Ubuntu
1. Install Solaar from github
```
sudo add-apt-repository ppa:solaar-unifying/stable
sudo apt-get update
```
2. Swap fn-mode via solar
```solaar config k400 fn-swap False```

***

## Удалить запись из EFI boot loader
1. Run a cmd.exe process with administrator privileges
1. Run `diskpart`
    + Type: list disk then sel disk X where X is the drive your boot files reside on
    + Type list vol to see all partitions (volumes) on the disk (the EFI volume will be formatted in FAT, others will be NTFS)
    + Select the EFI volume by typing: sel vol Y where Y is the SYSTEM volume (this is almost always the EFI partition)
    + For convenience, assign a drive letter by typing: assign letter=Z: where Z is a free (unused) drive letter
    + Type exit to leave disk part

1. While still in the cmd prompt, type: Z: and hit enter, where Z was the drive letter you just created.
    + Type dir to list directories on this mounted EFI partition
    + If you are in the right place, you should see a directory called EFI
    + Type cd EFI and then dir to list the child directories inside EFI
    + Type rmdir /S ubuntu to delete the ubuntu boot directory

1. Install efibootmgr `sudo apt-get install efibootmgr`
1. `sudo modprobe efivars`
1. `sudo efibootmgr`
1. `sudo efibootmgr -b 5 -B`

***

## Terminal

### Kill process by name
```
pkill python
```

### Kill process by pid
```
kill 9476
```

### Search process by port
```
lsof -i -P -n | grep LISTEN | grep 8888
```

### redirect 80 to 8888 via iptables
```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8888 && sudo iptables -t nat -I OUTPUT -p tcp -d 127.0.0.1 --dport 80 -j REDIRECT --to-ports 8888
```

### Multi-ping
```
#!/bin/bash
for i in {1..100}
do
	bash -c "ping 10.55.56.110 -i 0.200 -c 1000 -s 1000;" &
done
```

### Rsync
```
rsync -ave "ssh -p 5022" user@10.10.0.2:/mnt/media/some-project/logs/old_logs/ /home/user/backups/logs

rsync --remove-source-files -m --include='*.back.7z' --exclude='*' -ave "ssh -p 5022" user@10.10.0.2:/data/backup/ /home/user/backups/Database

rsync --delete -ave "ssh -p 5022" user@10.10.0.2:/mnt/media /home/user/backups/Media
```

### Redis-cli
```
redis-cli -h 10.10.1.235 -p 6379 -a SeCrEt
```

### Dates

#### Current unix-time
```
date +%s
```

#### Date in format 202004091032
```
date +"%Y%m%d%H%M"
```
