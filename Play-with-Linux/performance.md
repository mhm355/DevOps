## 1. Install and Configure zRAM using `zram-tools`

```bash
sudo apt update

sudo apt install zram-tools

sudo nano /etc/defaults/zramswap  #change the `ALGO` to be `zstd`, `PERCENT` to be `60` and `PRIORITY` to be `100`

sudo systemctl reload zramswap  # reload the service to read the new chnages in config file

zramctl  # check that the service is running

# to show the priority 

swapon --show
```

 
* the configuration file `/etc/defaults/zramswap`

```unit
# Compression algorithm selection
# speed: lz4 > zstd > lzo
# compression: zstd > lzo > lz4
# This is not inclusive of all that is available in latest kernels
# See /sys/block/zram0/comp_algorithm (when zram module is loaded) to see
# what is currently set and available for your kernel[1]
# [1]  https://github.com/torvalds/linux/blob/master/Documentation/blockdev/zram.txt#L86
#ALGO=lz4

# Specifies the amount of RAM that should be used for zram
# based on a percentage the total amount of available memory
# This takes precedence and overrides SIZE below
#PERCENT=50

# Specifies a static amount of RAM that should be used for
# the ZRAM devices, this is in MiB
#SIZE=256

# Specifies the priority for the swap devices, see swapon(2)
# for more details. Higher number = higher priority
# This should probably be higher than hdd/ssd swaps.
#PRIORITY=100
```

* change the `ALGO` to be `zstd`, `PERCENT` to be `60` and `PRIORITY` to be `100` 


> 1. speed: lz4 > zstd > lzo

* This line ranks them by how fast they can compress/decompress data.

* lz4 (The Sprinter): It is the fastest. It uses very little CPU power. It effectively adds RAM without slowing down your laptop at all.

* zstd (The Modern Standard): It is fast enough, but slower than lz4. It takes a split second longer to process data.

* lzo: An older standard. In this specific user's list, they rank it as slowest, though in reality, it often sits between lz4 and zstd depending on the version.

> 2. compression: zstd > lzo > lz4

* This line ranks them by how much space they save (how tiny they can squash the data).

* zstd (The Crusher): The clear winner. It can squeeze data much tighter than the others. If you have 1GB of data, zstd might squeeze it down to 300MB, whereas lz4 might only get it to 500MB.

* lzo: Middle of the road.

* lz4: The "loosest" compression. It doesn't squeeze as hard because it's focused on speed.


## 2. Install zRAM using `zram-config`

* using about 50% of your RAM for the compressed block

```bash

sudo apt install zram-config

systemctl restart

zramctl
```




## 3. Adjusting Swappiness

* Swap (Virtual Memory): This is a file on your hard drive. It acts as "overflow" memory. When your RAM gets full, the computer moves inactive data here to prevent a crash. Swap is much slower than RAM.

* `Swappiness` is  a setting (a number between 0 and 100) that tells computer how eager it should be to move data from the fast RAM to the slow Swap

* High Swappiness (e.g., 60-100): The computer moves data to the hard drive very often, even if there is still space in RAM.

* Low Swappiness (e.g., 1-10): The computer tries to keep data in the super-fast RAM for as long as possible and only touches the hard drive when absolutely necessary.

* kepp it high value and enable zram 

```bash

cat /proc/sys/vm/swappiness 

sudo nano /etc/sysctl.conf  # add this line  vm.swappiness=20

sudo sysctl -p    # sysctl command used to view and modify kernel parameters at runtime.

```


## 4. VFS Cache Pressure 

* (vm.vfs_cache_pressure) is a Linux kernel parameter (default 100) that controls how aggressively the kernel reclaims memory used for directory (dentry) and inode caches,

* it controls

    * Dentry Cache (dcache): Stores directory entries (names to inodes).
    * Inode Cache (icache): Stores inode information (file metadata).* 


* Default (100): A balanced approach where dentries and inodes are reclaimed at the same rate as page cache.
* < 100 (e.g., 50, 10): The kernel prioritizes keeping dentry/inode caches, reducing disk lookups but using more RAM for caching.
* `>` 100 (e.g., 200, 1000): The kernel aggressively frees dentry/inode memory, making room for page cache but potentially slowing down file access.
* 0: The kernel never reclaims these caches due to memory pressure, risking system instability. 

* Increase (e.g., to 200+): For systems with huge numbers of files where VFS caches grow too large and starve applications of RAM.
* Decrease (e.g., to 50-80): For databases or file servers where many small files are accessed frequently, benefiting from keeping metadata in memory. 


* decrease it for `HDD` with zram 

    * keep the map of files in the fast RAM. If you need space, move the actual file contents to zram, but do not make me search the slow mechanical disk for the file map again."

```bash

# at runtime 
sudo sysctl vm.vfs_cache_pressure=50 # set the value 

cat /proc/sys/vm/vfs_cache_pressure   # show the value assigned

# permanent

sudo nano /etc/sysctl.conf  # add this line vm.vfs_cache_pressure=50
```


# 5. The "Observer Effect" (noatime)

* The Problem: You are asking the computer to read, but it is forced to write. On a slow mechanical HDD, this causes the read/write needle to jump around unnecessarily. It wears out the drive and adds a tiny delay to everything.

* The Solution: noatime We want to tell the system: "I don't care when a file was last read. Just let me read it without writing anything."

* default is `relatime`

```bash

findmnt /  # check tha status

sudo cp /etc/fstab /etc/fstab.backup

sudo nano /etc/fstab   # chnage it to defaults,noatime and can assign it to /home

sudo mount -a # check if no output the syntax is correct

sudo systemctl restart
```


# 6. Fixing "System Freezes" (Dirty Bytes)

