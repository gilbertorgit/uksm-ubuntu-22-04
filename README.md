# Ubuntu 22.04 with UKSM 

## Authors

* **Gilberto**

## Optional Config - UKSM 

* Although it's totally optional but as you can se below, it likely to improve your server performance as well as enable some additional kernel features. 

```
#memory use before
root@ubuntu:/home/lab/apstra_script# free -g
              total        used        free      shared  buff/cache   available
Mem:            125          77          31           0          17          47
Swap:             0           0           0


#memory use after USKM 
root@ubuntu:/home/lab/apstra_script# free -g
              total        used        free      shared  buff/cache   available
Mem:            125          36          70           0          18          86
Swap:             0           0           0
```

```
apt -y install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc gnupg2 libelf-dev flex  bison dwarves zstd libncurses-dev 

git clone https://github.com/dolohow/uksm.git

# verify, acho que nao precisa disso
apt -y --no-install-recommends install linux-source-5.15.0 

```


```
git clone https://github.com/dolohow/uksm.git
git clone --depth 1 --single-branch --branch Ubuntu-5.15.0-75.82 https://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/jammy

cd jammy/
patch -p1 < ../uksm/v5.x/uksm-5.15.patch 
```

**you will get a log like the one below:** 
```
###### Log Example:
root@jerry-kvm:~/jammy# patch -p1 < ../uksm/v5.x/uksm-5.15.patch 
patching file Documentation/vm/uksm.txt
patching file fs/exec.c
patching file fs/proc/meminfo.c
patching file include/linux/ksm.h
patching file include/linux/mm_types.h
Hunk #1 succeeded at 387 (offset 2 lines).
patching file include/linux/mmzone.h
patching file include/linux/pgtable.h
Hunk #1 succeeded at 1138 (offset 1 line).
Hunk #2 succeeded at 1166 (offset 1 line).
patching file include/linux/sradix-tree.h
patching file include/linux/uksm.h
patching file kernel/fork.c
Hunk #1 succeeded at 612 (offset 8 lines).
patching file lib/Makefile
patching file lib/sradix-tree.c
patching file mm/Kconfig
patching file mm/Makefile
patching file mm/ksm.c
patching file mm/memory.c
Hunk #4 succeeded at 1382 (offset 11 lines).
Hunk #5 succeeded at 2794 (offset 14 lines).
Hunk #6 succeeded at 3041 (offset 14 lines).
Hunk #7 succeeded at 3084 (offset 14 lines).
patching file mm/mmap.c
Hunk #2 succeeded at 186 with fuzz 2.
Hunk #8 succeeded at 1865 (offset 4 lines).
Hunk #9 succeeded at 1907 (offset 6 lines).
Hunk #10 succeeded at 2777 (offset 15 lines).
Hunk #11 succeeded at 3086 (offset 32 lines).
Hunk #12 succeeded at 3132 (offset 32 lines).
Hunk #13 succeeded at 3209 (offset 32 lines).
Hunk #14 succeeded at 3244 (offset 32 lines).
Hunk #15 succeeded at 3356 (offset 32 lines).
Hunk #16 succeeded at 3521 (offset 32 lines).
patching file mm/uksm.c
patching file mm/vmstat.c
```

```
- edit br_private.h and change from
vi net/bridge/br_private.h
#define BR_GROUPFWD_RESTRICTED (BR_GROUPFWD_STP | BR_GROUPFWD_MACPAUSE | \
                                BR_GROUPFWD_LACP)

- to
vi net/bridge/br_private.h
#define BR_GROUPFWD_RESTRICTED  0x0000u
```

```
make oldconfig
```

```
scripts/config --disable DEBUG_INFO
sed -i 's/CONFIG_SYSTEM_TRUSTED_KEYS="debian\/canonical-certs.pem"/CONFIG_SYSTEM_TRUSTED_KEYS=""/g' .config
sed -i 's/CONFIG_SYSTEM_REVOCATION_KEYS="debian\/canonical-revoked-certs.pem"/CONFIG_SYSTEM_REVOCATION_KEYS=""/g' .config
cat .config | grep _KEYS

you run this one instead:

scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS

```

```
make -j$(nproc) deb-pkg LOCALVERSION=-uksm

ls -la ../*deb
-rw-r--r-- 1 root root  8766728 Jun 15 18:32 ../linux-headers-5.15.99-uksm_5.15.99-uksm-1_amd64.deb
-rw-r--r-- 1 root root 84156016 Jun 15 18:34 ../linux-image-5.15.99-uksm_5.15.99-uksm-1_amd64.deb
-rw-r--r-- 1 root root  1247446 Jun 15 18:32 ../linux-libc-dev_5.15.99-uksm-1_amd64.deb
```


```
dpkg -i ../linux-headers-5.15.99-uksm_5.15.99-uksm-1_amd64.deb
dpkg -i ../linux-image-5.15.99-uksm_5.15.99-uksm-1_amd64.deb

update-grub

shutdown -r now
```


```
uname -a
Linux jerry-kvm 5.15.99-uksm #1 SMP Thu Jun 15 18:22:50 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

```
GRUB_CMDLINE_LINUX="mitigations=off default_hugepagesz=1G hugepagesz=1G hugepages=64G"
```



