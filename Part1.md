# Part I : A bit of exploration

## 1. /proc

🌞 **Afficher...** :

- l'état complet de la mémoire (RAM)

```
[dash@localhost ~]$ cat /proc/meminfo 
MemTotal:         468908 kB
MemFree:          209460 kB
MemAvailable:     311884 kB
Buffers:            2708 kB
Cached:            86572 kB
SwapCached:            0 kB
Active:           129000 kB
Inactive:          18128 kB
Active(anon):      57788 kB
Inactive(anon):     2624 kB
Active(file):      71212 kB
Inactive(file):    15504 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:        839676 kB
SwapFree:         839676 kB
Zswap:                 0 kB
Zswapped:              0 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:         57888 kB
Mapped:            39832 kB
Shmem:              2564 kB
KReclaimable:      27728 kB
Slab:              62428 kB
SReclaimable:      27728 kB
SUnreclaim:        34700 kB
KernelStack:        3232 kB
PageTables:         1588 kB
SecPageTables:         0 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     1074128 kB
Committed_AS:     203360 kB
VmallocTotal:   34359738367 kB
VmallocUsed:       22676 kB
VmallocChunk:          0 kB
Percpu:              384 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
FilePmdMapped:         0 kB
CmaTotal:              0 kB
CmaFree:               0 kB
Unaccepted:            0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:       98176 kB
DirectMap2M:      425984 kB
DirectMap1G:           0 kB 
```

- le nombre de coeurs que votre CPU a (uniquement ce nombre)
```
[dash@localhost ~]$ cat /proc/cpuinfo | grep "cpu cores"
cpu cores	: 1
```

- le nombre de processus lancés (uniquement ce nombre)
```
[dash@localhost ~]$ cat /proc/loadavg | awk '{print $4}' | cut -d '/' -f2
202
```

- la ligne de commande utilisée pour lancer le kernel actuel
```
[dash@localhost ~]$ cat /proc/cmdline
BOOT_IMAGE=(hd0,msdos1)/vmlinuz-5.14.0-427.13.1.el9_4.x86_64 root=/dev/mapper/rl-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rl-swap rd.lvm.lv=rl/root rd.lvm.lv=rl/swap
```

- la liste des connexions TCP actuelles (même si c'est un peu imbuvable avec nos p'tits yeux)
```
[dash@localhost ~]$ cat /proc/net/tcp
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode                                                     
   0: 00000000:0016 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 20352 1 0000000000000000 100 0 0 10 0                     
   1: 8485A8C0:0016 8085A8C0:EA46 01 00000000:00000000 02:00092BB6 00000000     0        0 21338 4 0000000000000000 20 6 21 10 20                    
```

- la valeur actuelle de la *swappiness* (cf le tip ci-dessous)
```
[dash@localhost ~]$ cat /proc/sys/vm/swappiness 
60
```

## 2. /sys

> N'oubliez jamais les pages du `man`, c'est un très bonne doc souvent. [Là encore pour **sysfs** (`/sys`)](https://man7.org/linux/man-pages/man5/sysfs.5.html).

🌞 **Afficher...** :

- la liste des périphériques de types bloc reconnus par l'OS (genre les disques durs par exemple koa)
```
[dash@localhost ~]$ ls /sys/block/
dm-0  dm-1  sda
```

- la liste des modules kernel qui sont actuellement en cours d'utilisation
```
[dash@localhost ~]$ ls /sys/module
8250              crc64_rocksoft    efi_pstore           intel_rapl_common  mousedev        nft_reject       rfkill        sysrq                  virtio_pci_modern_dev
acpi              crc_t10dif        efivars              intel_rapl_msr     msr             nft_reject_inet  rng_core      t10_pi                 vmd
acpiphp           crct10dif_pclmul  ehci_hcd             ip_set             netpoll         nmi_backtrace    rtc_cmos      tcp_cubic              vmw_balloon
ahci              cryptomgr         fb                   ipv6               nf_conntrack    page_alloc       scsi_dh_alua  thermal                vmwgfx
ata_generic       damon_reclaim     fb_sys_fops          joydev             nf_defrag_ipv4  page_reporting   scsi_dh_rdac  thunderbolt            vmw_vmci
ata_piix          debug_core        firmware_class       kdb                nf_defrag_ipv6  pcie_aspm        scsi_mod      tpm                    vt
battery           device_hmem       fuse                 kernel             nf_nat          pciehp           sd_mod        tpm_crb                watchdog
blk_cgroup        dm_log            ghash_clmulni_intel  keyboard           nfnetlink       pci_hotplug      secretmem     tpm_tis                workqueue
block             dm_mirror         gpiolib_acpi         kfence             nf_reject_ipv4  pcmcia_core      serio_raw     tpm_tis_core           xen
button            dm_mod            haltpoll             kgdboc             nf_reject_ipv6  pcspkr           sg            ttm                    xfs
clocksource       dm_region_hash    hid                  kgdbts             nf_tables       printk           shpchp        udmabuf                xhci_hcd
configfs          drm               hid_magicmouse       libahci            nft_chain_nat   processor        spurious      uhci_hcd               xz_dec
cpufreq           drm_kms_helper    hid_ntrig            libata             nft_ct          psmouse          srcutree      usbcore                zswap
cpuidle           drm_ttm_helper    i2c_piix4            libcrc32c          nft_fib         pstore           suspend       usbhid
cpuidle_haltpoll  dynamic_debug     i8042                md_mod             nft_fib_inet    random           syscopyarea   uv_nmi
crc32c_intel      e1000             ima                  memory_hotplug     nft_fib_ipv4    rcupdate         sysfillrect   virtio_pci
crc32_pclmul      edac_core         intel_idle           module             nft_fib_ipv6    rcutree          sysimgblt     virtio_pci_legacy_dev
```

- la liste des cartes réseau

```
[dash@localhost ~]$ ls /sys/class/net
ens32  ens33  lo
```
