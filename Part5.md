🌞 **Proposez un nouveau fichier `web.service`**

```
[dash@localhost ~]$ sudo useradd -r -s /sbin/nologin webuser
[sudo] password for dash: 
[dash@localhost ~]$ sudo mkdir -p /var/www/webroot
[dash@localhost ~]$ sudo touch /var/www/webroot/test.html
[dash@localhost ~]$ sudo chown -R webuser:webuser /var/www/webroot
```
- Configuration du fichier `web.service`
```
[dash@localhost ~]$ sudo nano /etc/systemd/system/web.service
[Unit]
Description=Hardened Web Server
After=network.target

[Service]
User=webuser
Group=webuser
ExecStart=/usr/bin/python3 -m http.server 9999

# Chroot d'un repertoire spécifique
WorkingDirectory=/var/www/webroot
ReadOnlyDirectories=/var/www/webroot

#Isolation de processus avec des namespaces
PrivateTmp=yes
PrivateDevices=yes
RestrictNamespaces=yes
RemoveIPC=yes
PrivateMounts=yes

# Restriction des syscalls
NoNewPrivileges=yes
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
SecureBits=no-setuid-fixup-locked
SystemCallFilter=@system-service
SystemCallErrorNumber=EPERM

#Limitation des ressources
MemoryMax=321M
CPUQuota=50%
IOWriteBandwidthMax=/dev/sda 1M
IOReadBandwidthMax=/dev/sda 1M
LimitNOFILE=1000
LimitNPROC=50

#Protection du système et des données sensibles
ProtectSystem=strict
ProtectHome=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
LockPersonality=yes
RestrictRealtime=yes
RestrictSUIDSGID=yes
ProtectClock=yes
ProtectHostname=yes
ProtectKernelLogs=yes
ProtectProc=invisible
ProcSubset=pid
UMask=0077

#Restrictions réseau
RestrictAddressFamilies=AF_INET AF_INET6

[Install]
WantedBy=multi-user.target
```
- Le service web fonctionne bien
```
[dash@localhost ~]$ sudo systemctl daemon-reload
[dash@localhost ~]$ sudo systemctl restart web.service
[dash@localhost ~]$ curl localhost:9999
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="test.html">test.html</a></li>
</ul>
<hr>
</body>
</html>

```
- On peut se rendre compte que notre config est plus sécurisée
```
[dash@localhost ~]$ systemd-analyze security web.service
  NAME                                                        DESCRIPTION                                                                                         EXPOSURE
✓ SystemCallFilter=~@swap                                     System call allow list defined for service, and @swap is not included                                       
✗ SystemCallFilter=~@resources                                System call allow list defined for service, and @resources is included (e.g. ioprio_set is allowed)      0.2
✓ SystemCallFilter=~@reboot                                   System call allow list defined for service, and @reboot is not included                                     
✓ SystemCallFilter=~@raw-io                                   System call allow list defined for service, and @raw-io is not included                                     
✗ SystemCallFilter=~@privileged                               System call allow list defined for service, and @privileged is included (e.g. chown is allowed)          0.2
✓ SystemCallFilter=~@obsolete                                 System call allow list defined for service, and @obsolete is not included                                   
✓ SystemCallFilter=~@mount                                    System call allow list defined for service, and @mount is not included                                      
✓ SystemCallFilter=~@module                                   System call allow list defined for service, and @module is not included                                     
✓ SystemCallFilter=~@debug                                    System call allow list defined for service, and @debug is not included                                      
✓ SystemCallFilter=~@cpu-emulation                            System call allow list defined for service, and @cpu-emulation is not included                              
✓ SystemCallFilter=~@clock                                    System call allow list defined for service, and @clock is not included                                      
✓ RemoveIPC=                                                  Service user cannot leave SysV IPC objects around                                                           
✗ RootDirectory=/RootImage=                                   Service runs within the host's root directory                                                            0.1
✓ User=/DynamicUser=                                          Service runs under a static non-root user identity                                                          
✓ RestrictRealtime=                                           Service realtime scheduling access is restricted                                                            
✓ CapabilityBoundingSet=~CAP_SYS_TIME                         Service processes cannot change the system clock                                                            
✓ NoNewPrivileges=                                            Service processes cannot acquire new privileges                                                             
✗ AmbientCapabilities=                                        Service process receives ambient capabilities                                                            0.1
✗ SystemCallArchitectures=                                    Service may execute system calls with all ABIs                                                           0.2
✗ MemoryDenyWriteExecute=                                     Service may create writable executable memory mappings                                                   0.1
✗ RestrictAddressFamilies=~AF_(INET|INET6)                    Service may allocate Internet sockets                                                                    0.3
✓ ProtectSystem=                                              Service has strict read-only access to the OS file hierarchy                                                
✓ ProtectProc=                                                Service has restricted access to process tree (/proc hidepid=)                                              
✓ SupplementaryGroups=                                        Service has no supplementary groups                                                                         
✓ CapabilityBoundingSet=~CAP_SYS_RAWIO                        Service has no raw I/O access                                                                               
✓ CapabilityBoundingSet=~CAP_SYS_PTRACE                       Service has no ptrace() debugging abilities                                                                 
✓ CapabilityBoundingSet=~CAP_SYS_(NICE|RESOURCE)              Service has no privileges to change resource use parameters                                                 
✓ CapabilityBoundingSet=~CAP_NET_ADMIN                        Service has no network configuration privileges                                                             
✓ CapabilityBoundingSet=~CAP_AUDIT_*                          Service has no audit subsystem access                                                                       
✓ CapabilityBoundingSet=~CAP_SYS_ADMIN                        Service has no administrator privileges                                                                     
✓ PrivateTmp=                                                 Service has no access to other software's temporary files                                                   
✓ ProcSubset=                                                 Service has no access to non-process /proc files (/proc subset=)                                            
✓ CapabilityBoundingSet=~CAP_SYSLOG                           Service has no access to kernel logging                                                                     
✓ ProtectHome=                                                Service has no access to home directories                                                                   
✓ PrivateDevices=                                             Service has no access to hardware devices                                                                   
✗ CapabilityBoundingSet=~CAP_NET_(BIND_SERVICE|BROADCAST|RAW) Service has elevated networking privileges                                                               0.1
✗ PrivateNetwork=                                             Service has access to the host's network                                                                 0.5
✗ PrivateUsers=                                               Service has access to other users                                                                        0.2
✗ DeviceAllow=                                                Service has a device ACL with some special devices: char-rtc:r                                           0.1
✓ KeyringMode=                                                Service doesn't share key material with other services                                                      
✓ Delegate=                                                   Service does not maintain its own delegated control group subtree                                           
✗ IPAddressDeny=                                              Service does not define an IP address allow list                                                         0.2
✓ NotifyAccess=                                               Service child processes cannot alter service state                                                          
✓ ProtectClock=                                               Service cannot write to the hardware clock or system clock                                                  
✓ CapabilityBoundingSet=~CAP_SYS_PACCT                        Service cannot use acct()                                                                                   
✓ CapabilityBoundingSet=~CAP_KILL                             Service cannot send UNIX signals to arbitrary processes                                                     
✓ ProtectKernelLogs=                                          Service cannot read from or write to the kernel log ring buffer                                             
✓ CapabilityBoundingSet=~CAP_WAKE_ALARM                       Service cannot program timers that wake up the system                                                       
✓ CapabilityBoundingSet=~CAP_(DAC_*|FOWNER|IPC_OWNER)         Service cannot override UNIX file/IPC permission checks                                                     
✓ ProtectControlGroups=                                       Service cannot modify the control group file system                                                         
✓ CapabilityBoundingSet=~CAP_LINUX_IMMUTABLE                  Service cannot mark files immutable                                                                         
✓ CapabilityBoundingSet=~CAP_IPC_LOCK                         Service cannot lock memory into RAM                                                                         
✓ ProtectKernelModules=                                       Service cannot load or read kernel modules                                                                  
✓ CapabilityBoundingSet=~CAP_SYS_MODULE                       Service cannot load kernel modules                                                                          
✓ CapabilityBoundingSet=~CAP_SYS_TTY_CONFIG                   Service cannot issue vhangup()                                                                              
✓ CapabilityBoundingSet=~CAP_SYS_BOOT                         Service cannot issue reboot()                                                                               
✓ CapabilityBoundingSet=~CAP_SYS_CHROOT                       Service cannot issue chroot()                                                                               
✓ PrivateMounts=                                              Service cannot install system mounts                                                                        
✓ CapabilityBoundingSet=~CAP_BLOCK_SUSPEND                    Service cannot establish wake locks                                                                         
✓ RestrictNamespaces=~user                                    Service cannot create user namespaces                                                                       
✓ RestrictNamespaces=~pid                                     Service cannot create process namespaces                                                                    
✓ RestrictNamespaces=~net                                     Service cannot create network namespaces                                                                    
✓ RestrictNamespaces=~uts                                     Service cannot create hostname namespaces                                                                   
✓ RestrictNamespaces=~mnt                                     Service cannot create file system namespaces                                                                
✓ CapabilityBoundingSet=~CAP_LEASE                            Service cannot create file leases                                                                           
✓ CapabilityBoundingSet=~CAP_MKNOD                            Service cannot create device nodes                                                                          
✓ RestrictNamespaces=~cgroup                                  Service cannot create cgroup namespaces                                                                     
✓ RestrictNamespaces=~ipc                                     Service cannot create IPC namespaces                                                                        
✓ ProtectHostname=                                            Service cannot change system host/domainname                                                                
✓ CapabilityBoundingSet=~CAP_(CHOWN|FSETID|SETFCAP)           Service cannot change file ownership/access mode/capabilities                                               
✓ CapabilityBoundingSet=~CAP_SET(UID|GID|PCAP)                Service cannot change UID/GID identities/capabilities                                                       
✓ LockPersonality=                                            Service cannot change ABI personality                                                                       
✓ ProtectKernelTunables=                                      Service cannot alter kernel tunables (/proc/sys, …)                                                         
✓ RestrictAddressFamilies=~AF_PACKET                          Service cannot allocate packet sockets                                                                      
✓ RestrictAddressFamilies=~AF_NETLINK                         Service cannot allocate netlink sockets                                                                     
✓ RestrictAddressFamilies=~AF_UNIX                            Service cannot allocate local sockets                                                                       
✓ RestrictAddressFamilies=~…                                  Service cannot allocate exotic sockets                                                                      
✓ CapabilityBoundingSet=~CAP_MAC_*                            Service cannot adjust SMACK MAC                                                                             
✓ RestrictSUIDSGID=                                           SUID/SGID file creation by service is restricted                                                            
✓ UMask=                                                      Files created by service are accessible only by service's own user by default                               

→ Overall exposure level for web.service: 1.8 OK 🙂
```
