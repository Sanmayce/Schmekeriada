# Schmekerezada - superfast Linux/Windows console tool to sort lines, internally
Homepage: http://www.sanmayce.com/Schmekeriada/

In the tar package there are full C sourcecode and the speed showdown logs between sort (GNU coreutils) 9.1 and Schmekerezada_(2023-Jul-10).
In short, when sorting in-memory (i.e. not stressing virtual memory), 'Schmekerezada' is faster than 'sort'.

![Schmekerezada](https://github.com/Sanmayce/Schmekeriada/assets/14062548/0e499126-d963-4fc7-8eaa-6f5b5fe03c2e)

One of the benchmarks - sorting Linux kernel 38.9/33.1=1.17x faster than 'sort' on a "weak-n-old" educational laptop with Celeron N4100:

```
Machine:
Laptop Thinkpad 11e gen5, 4 cores/threads.

Partition in use:
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/nvme0n1p2 ext4  875G  567G  264G  69% /

Locale:
LC_ALL=C

 Performance counter stats for './Schmekerezada_CLANG_16.0.1_SSE4.2_TetraThread.elf linux-6.1.38.tar':

          60188.79 msec task-clock                       #    1.815 CPUs utilized             
              2367      context-switches                 #   39.326 /sec                      
                20      cpu-migrations                   #    0.332 /sec                      
            733279      page-faults                      #   12.183 K/sec                     
      136488603825      cycles                           #    2.268 GHz                         (57.12%)
       65342898585      instructions                     #    0.48  insn per cycle              (71.42%)
       13854546733      branches                         #  230.185 M/sec                       (71.42%)
         442781547      branch-misses                    #    3.20% of all branches             (71.44%)
       14868989544      L1-dcache-loads                  #  247.039 M/sec                       (71.45%)
   <not supported>      L1-dcache-load-misses                                                 
        1124502711      LLC-loads                        #   18.683 M/sec                       (71.44%)
         433726013      LLC-load-misses                  #   38.57% of all LL-cache accesses    (57.13%)

      33.153667384 seconds time elapsed

      52.500108000 seconds user
       6.000191000 seconds sys

 Performance counter stats for 'sort -o Linuxsort linux-6.1.38.tar --parallel=4 -T ./':

          72840.58 msec task-clock                       #    1.871 CPUs utilized             
              5032      context-switches                 #   69.082 /sec                      
               125      cpu-migrations                   #    1.716 /sec                      
            768074      page-faults                      #   10.545 K/sec                     
      165111578337      cycles                           #    2.267 GHz                         (57.14%)
      102796467972      instructions                     #    0.62  insn per cycle              (71.43%)
       21511871163      branches                         #  295.328 M/sec                       (71.41%)
         451366228      branch-misses                    #    2.10% of all branches             (71.43%)
       29825878712      L1-dcache-loads                  #  409.468 M/sec                       (71.44%)
   <not supported>      L1-dcache-load-misses                                                 
        1296854629      LLC-loads                        #   17.804 M/sec                       (71.44%)
         276985676      LLC-load-misses                  #   21.36% of all LL-cache accesses    (57.14%)

      38.933277562 seconds time elapsed

      55.537844000 seconds user
      15.226412000 seconds sys
```

The full benchmark script, 'bench_PARAMETER.sh':
```
#sudo mkdir /tmp/ramdisk
#sudo chmod 777 /tmp/ramdisk
#sudo mount -t tmpfs -o size=32G myramdisk /tmp/ramdisk
#sudo umount /tmp/ramdisk/

cat /proc/version
echo
sudo fdisk -l
echo
lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,MODEL,LABEL
echo
df -h -T
echo
echo Partition in use:
df -hT .
echo
echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo
lscpu
echo
free -h
echo
#[root@kaze Schmekerezada]# swapon -a
#[root@kaze Schmekerezada]# swapon --show
#NAME      TYPE        SIZE USED PRIO
#/dev/sdb2 partition 128.8G   0B   -2
#[root@kaze Schmekerezada]# swapoff -a
#[root@kaze Schmekerezada]# swapon --show
#[root@kaze Schmekerezada]# 

echo Enabling swap...
swapon -a
echo Swap status:
swapon --show
echo
export LC_ALL='C'
locale
echo $LC_ALL

cp "$1" /dev/null
date +%T
perf stat -d ./Schmekerezada_CLANG_16.0.1_SSE4.2_TetraThread.elf "$1"
sha1sum Schmekeriada.txt 
perf stat -d ./Schmekerezada_GCC_13.0.1_SSE4.2_TetraThread.elf "$1"
date +%T
#/bin/time -v ./Schmekeriada_GCC_12.1.1_TetraThread.elf "$1"
sort --version
date +%T
perf stat -d sort -o Linuxsort "$1" --parallel=4 -T ./
date +%T
#/bin/time -v sort -o Linuxsort "$1" --parallel=4 -T ./
sha1sum Schmekeriada.txt 
sha1sum Linuxsort
date +%T
perf stat -d sort -o Linuxsort "$1" --parallel=16 -T ./
date +%T
#/bin/time -v sort -o Linuxsort "$1" --parallel=16 -T ./
rm Schmekeriada.txt 
rm Linuxsort
```

On SATA SSD it is even faster, up to 44%:
```
sort_vs_Schmekerezada.sh:

#   _________        .__                       __                                               .___        
#  /   _____/  ____  |  |__    _____    ____  |  | __  ____ _______   ____  _____________     __| _/_____   
#  \_____  \ _/ ___\ |  |  \  /     \ _/ __ \ |  |/ /_/ __ \\_  __ \_/ __ \ \___   /\__  \   / __ | \__  \  
#  /        \\  \___ |   Y  \|  Y Y  \\  ___/ |    < \  ___/ |  | \/\  ___/  /    /  / __ \_/ /_/ |  / __ \_
# /_______  / \___  >|___|  /|__|_|  / \___  >|__|_ \ \___  >|__|    \___  >/_____ \(____  /\____ | (____  /
#         \/      \/      \/       \/      \/      \/     \/             \/       \/     \/      \/      \/ 

if [ ! -f "./30000000.KnightTours.txt" ]; then
sh GENERATE_Xmillion_Knight-Tours.sh 30000000
fi
sh bench_PARAMETER.sh linux-6.1.38.tar
sh bench_PARAMETER.sh www.ncbi.nlm.nih.gov_genome_guide_human_GRCh38_latest_genomic.fna
sh bench_PARAMETER.sh Fedora-Workstation-Live-x86_64-38-1.6.iso
sh bench_PARAMETER.sh 30000000.KnightTours.txt

#-rwxrwxrwx. 1 kaze kaze 1359441920 Jul 11 14:53 linux-6.1.38.tar
#-rwxrwxrwx. 1 kaze kaze 3313061631 Apr  8  2022 www.ncbi.nlm.nih.gov_genome_guide_human_GRCh38_latest_genomic.fna
#-rwxrwxrwx. 1 kaze kaze 2099451904 Jul 11 17:35 Fedora-Workstation-Live-x86_64-38-1.6.iso
#-rwxrwxrwx. 1 kaze kaze 3870000000 Jul 12 01:33 30000000.KnightTours.txt

# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# |        \ Corpus Name             |              linux-6.1.38.tar |              Human_Genome_DNA |      30000000.KnightTours.txt | Fedora-Workstat...-38-1.6.iso |
# |        \ Corpus Size in Bytes    |                 1,359,441,920 |                 3,313,061,631 |                 3,870,000,000 |                 2,099,451,904 |
# | Sorter \ Corpus Size in Lines    |                    35,585,653 |                    40,902,071 |                    30,000,000 |                     8,189,810 |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | sort v9.0 --parallel=4 -T ./     |  99,472,516,848 instructions  | 128,285,206,805 instructions  | 109,679,415,651 instructions  |  27,716,611,486 instructions  |
# |                                  |  21,494,037,021 branches      |  27,548,013,135 branches      |  23,469,594,117 branches      |   5,918,985,732 branches      |
# |                                  |     461,065,737 branch-misses |     579,823,078 branch-misses |     156,337,384 branch-misses |     166,284,942 branch-misses |
# |                                  |            20.5 seconds       |            42.9 seconds       |            29.8 seconds       |            13.0 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | sort v9.0 --parallel=16 -T ./    |  99,826,000,759 instructions  | 128,975,431,969 instructions  | 106,611,922,490 instructions  |  30,359,262,417 instructions  |
# |                                  |  21,496,860,784 branches      |  27,614,861,865 branches      |  22,799,055,675 branches      |   6,494,585,359 branches      |
# |                                  |     465,714,553 branch-misses |     587,152,024 branch-misses |     169,894,912 branch-misses |     176,038,949 branch-misses |
# |                                  |            20.1 seconds       |            39.7 seconds       |            30.1 seconds       |            13.4 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | Schmekerezada_CLANG_16.0.1 (v19) |  64,682,963,173 instructions  | 104,566,733,962 instructions  | 135,489,760,619 instructions  |  37,726,691,231 instructions  |
# |                                  |  13,770,680,866 branches      |  22,623,299,550 branches      |  31,235,409,067 branches      |   9,080,478,581 branches      |
# |                                  |     423,462,180 branch-misses |     571,367,377 branch-misses |     521,102,315 branch-misses |     120,282,275 branch-misses |
# |                                  |            14.6 seconds       |            30.5 seconds       |            24.3 seconds       |             9.8 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | Schmekerezada_GCC_13.0.1 (v19)   |  73,570,739,976 instructions  | 120,331,569,174 instructions  | 146,686,441,751 instructions  |  42,112,144,823 instructions  |
# |                                  |  14,652,384,005 branches      |  24,465,263,549 branches      |  33,075,459,615 branches      |   9,627,303,775 branches      |
# |                                  |     421,083,216 branch-misses |     568,237,768 branch-misses |     523,361,025 branch-misses |     121,180,507 branch-misses |
# |                                  |            19.5 seconds       |            37.8 seconds       |            32.3 seconds       |            14.7 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# Note1: The benchmark is in 'performance' mode as superuser;
# Note2: Linux version 5.18.15-200.fc36.x86_64 (mockbuild@bkernel01.iad2.fedoraproject.org) (gcc (GCC) 12.1.1 20220507 (Red Hat 12.1.1-1), GNU ld version 2.37-27.fc36) #1 SMP PREEMPT_DYNAMIC Sun Jul 31 21:30:34 UTC 2022
# Note3: Testmachine: Laptop i5-7200U CPU, 3.1GHz max turbo 2cores/4threads, L1d: 64 KiB (2 instances), L1i: 64 KiB (2 instances), L2: 512 KiB (2 instances), L3: 3 MiB (1 instance), 36GB DDR4 2133MT/s, running Fedora 36;
# Note4: Schmekerezada is tetrathreaded;
# Note5: The current drive: SSD SATA KINGSTON SKC6001024G (1GB cache);
# Note6a: Partition in use:
# Note6b: Filesystem     Type  Size  Used Avail Use% Mounted on
# Note6c: /dev/sdb1      ext4  331G  226G   89G  72% /
# Note7: LC_ALL=C locale was used for sort;
# Note8: After sorting, checking the sha1sum for both outputs - they all matched;
# Note9: CLANG compiler is significantly better than GCC, too many times;
# NoteA: The KT_30M corpus is of fixed-line size - 128 bytes each line - all lines unique, here the LittleEndian-To-BigEndian technique pays off;
# NoteB: The time statistics (wall clock) are reported by Linuxâ€™ perf;
# NoteC: It is worth mentioning that CLANG executable is executed before GCC counterpart, it means possible caching of the whole file is more likely for the latter.
# 
# So, cumulatively:
# 14.6+30.5+24.3+9.8=79.2 seconds
# 20.1+39.7+30.1+13.4=103.3 seconds
# Schmekerezada (compiled with CLANG) is only 103.3/79.2=1.30x or 30% faster than GNUsort 16threads, cold shower for those who (like me) underestimated Mergesort's parallelizability.

# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# |        \ Corpus Name             |              linux-6.1.38.tar |              Human_Genome_DNA |      30000000.KnightTours.txt | Fedora-Workstat...-38-1.6.iso |
# |        \ Corpus Size in Bytes    |                 1,359,441,920 |                 3,313,061,631 |                 3,870,000,000 |                 2,099,451,904 |
# | Sorter \ Corpus Size in Lines    |                    35,585,653 |                    40,902,071 |                    30,000,000 |                     8,189,810 |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | sort v9.1 --parallel=4 -T ./     | 102,796,467,972 instructions  | 145,940,013,705 instructions  | 134,526,463,714 instructions  |  32,039,863,819 instructions  |
# |                                  |  21,511,871,163 branches      |  30,368,703,735 branches      |  27,618,897,382 branches      |   6,671,955,880 branches      |
# |                                  |     451,366,228 branch-misses |     646,691,572 branch-misses |     248,678,541 branch-misses |     182,670,430 branch-misses |
# |                                  |            38.9 seconds       |            89.6 seconds       |           101.2 seconds       |            22.6 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | sort v9.1 --parallel=16 -T ./    |  95,974,435,959 instructions  | 139,181,029,596 instructions  | 121,021,375,460 instructions  |  43,584,980,865 instructions  |
# |                                  |  20,010,961,812 branches      |  28,921,319,248 branches      |  24,752,255,755 branches      |   9,001,685,334 branches      |
# |                                  |     457,525,469 branch-misses |     638,439,543 branch-misses |     252,280,650 branch-misses |     251,235,618 branch-misses |
# |                                  |            34.6 seconds       |            76.0 seconds       |            97.7 seconds       |            31.6 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | Schmekerezada_CLANG_16.0.1 (v19) |  65,342,898,585 instructions  | 106,570,792,202 instructions  | 137,387,012,432 instructions  |  38,298,101,683 instructions  |
# |                                  |  13,854,546,733 branches      |  22,916,875,802 branches      |  31,582,212,549 branches      |   9,150,687,078 branches      |
# |                                  |     442,781,547 branch-misses |     565,776,851 branch-misses |     518,222,887 branch-misses |     125,573,907 branch-misses |
# |                                  |            33.1 seconds       |            61.4 seconds       |            50.7 seconds       |            20.9 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# | Schmekerezada_GCC_13.0.1 (v19)   |  75,034,087,501 instructions  | 123,284,759,010 instructions  | 150,447,993,706 instructions  |  43,962,088,192 instructions  |
# |                                  |  14,933,728,465 branches      |  25,022,793,479 branches      |  33,785,964,900 branches      |   9,980,116,461 branches      |
# |                                  |     436,610,153 branch-misses |     566,297,480 branch-misses |     545,784,186 branch-misses |     127,345,641 branch-misses |
# |                                  |            35.2 seconds       |            65.4 seconds       |            52.7 seconds       |            23.3 seconds       |
# +----------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+
# Note1: The benchmark is in 'performance' mode as superuser;
# Note2: Linux version 6.2.12-300.fc38.x86_64 (mockbuild@54604edad16f4e818e702bda973f7473) (gcc (GCC) 13.0.1 20230401 (Red Hat 13.0.1-0), GNU ld version 2.39-9.fc38) #1 SMP PREEMPT_DYNAMIC Thu Apr 20 23:05:25 UTC 2023
# Note3: Testmachine: Laptop Thinkpad 11e, Celeron N4100 CPU, 2.4GHz max turbo 4cores/4threads, L1d: 96 KiB (4 instances), L1i: 128 KiB (4 instances), L2: 4 MiB (1 instance), 8GB DDR4 2400MT/s, running Fedora 38;
# Note4: Schmekerezada is tetrathreaded;
# Note5: The current drive: SSD nvme 1TB TS1TMTE400S (DRAM-less cache);
# Note6a: Partition in use:
# Note6b: Filesystem     Type  Size  Used Avail Use% Mounted on
# Note6c: /dev/nvme0n1p2 ext4  875G  567G  264G  69% /
# Note7: LC_ALL=C locale was used for sort;
# Note8: After sorting, checking the sha1sum for both outputs - they all matched;
# Note9: CLANG compiler is significantly better than GCC, too many times;
# NoteA: The KT_30M corpus is of fixed-line size - 128 bytes each line - all lines unique, here the LittleEndian-To-BigEndian technique pays off;
# NoteB: The time statistics (wall clock) are reported by Linuxâ€™ perf;
# NoteC: It is worth mentioning that CLANG executable is executed before GCC counterpart, it means possible caching of the whole file is more likely for the latter.
# 
# So, cumulatively:
# 33.1+61.4+50.7+20.9=166.1 seconds
# 34.6+76.0+97.7+31.6=239.9 seconds
# Schmekerezada (compiled with CLANG) is only 239.9/166.1=1.44x or 44% faster than GNUsort 16threads, cold shower for those who (like me) underestimated Mergesort's parallelizability.
```

The benefits (compared to Windows' sort and Linux' sort) are:
- 100% FREE sourcecode, no licenses and shenanigans;
- Faster than both, see 'log_su_Intel_Celeron_N4100_Cores-4.txt';
- Neither of both are really cross-platform, no binaries (counterparts) found on the net that are transparently usable;
- Good starting point/playground for C coders.

Enfun!  
2023-Jul-12,  
Sanmayce
