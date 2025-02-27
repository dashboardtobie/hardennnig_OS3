
# Part III : CGroup

## 1. Explore

Pour rappel : la configuration actuelle des *CGroups* est dispo dans `/sys/fs/cgroup`.

🌞 **Afficher...** :

- la liste des controllers *CGroups* dispos sur le système
```
[dash@localhost ~]$ cat /sys/fs/cgroup/cgroup.controllers 
cpuset cpu io memory hugetlb pids rdma misc
```

- la quantité de mémoire max que vous êtes autorisés à utiliser dans votre session utilisateur
```
[dash@localhost ~]$ cat /sys/fs/cgroup/memory.max
cat: /sys/fs/cgroup/memory.max: No such file or directory
```

- les noms de tous les *CGroups* créés
```
[dash@localhost ~]$ ls /sys/fs/cgroup/
cgroup.controllers      cgroup.stat             cpuset.cpus.isolated   dev-mqueue.mount  memory.reclaim  sys-fs-fuse-connections.mount  system.slice
cgroup.max.depth        cgroup.subtree_control  cpuset.mems.effective  init.scope        memory.stat     sys-kernel-config.mount        user.slice
cgroup.max.descendants  cgroup.threads          cpu.stat               io.stat           misc.capacity   sys-kernel-debug.mount
cgroup.procs            cpuset.cpus.effective   dev-hugepages.mount    memory.numa_stat  misc.current    sys-kernel-tracing.mount
```

## 2. Do it

🌞 **Créer un nouveau *CGroup*** :

- appelez-le `meow`
```
[dash@localhost ~]$ sudo mkdir /sys/fs/cgroup/meow
```
- activez les controllers `cpu` `cpuset` et `memory` s'ils ne le sont pas déjà
```
[dash@localhost ~]$ sudo echo "+cpu +cpuset +memory" | sudo tee /sys/fs/cgroup/meow/cgroup.subtree_control
+cpu +cpuset +memory
```

🌞 **Créer un nouveau sous-CGroup** :

```
[dash@localhost ~]$ sudo mkdir /sys/fs/cgroup/meow/task1
```

- prouvez que les controllers activés sur `meow` ont bien été hérités
```
[dash@localhost ~]$ cat /sys/fs/cgroup/meow/task1/cgroup.controllers 
cpuset cpu memory

```

🌞 **Mettez en place une limitation RAM**

- définissez une limite de 150M de RAM pour ce CGroup `task1`
```
[dash@localhost ~]$ echo $((150 * 1024 * 1024)) | sudo tee /sys/fs/cgroup/meow/task1/memory.max
[sudo] password for dash: 
157286400
```

🌞 **Prouvez que la limite est effective**

1. utilisez la commande `stress-ng` pour remplir la mémoire de la machine
```
[dash@localhost ~]$ stress-ng --vm 1 --vm-bytes 100%
stress-ng: info:  [15309] defaulting to a 1 day, 0 secs run per stressor
stress-ng: info:  [15309] dispatching hogs: 1 vm
```
2. constatez que la RAM est pleine
```
[dash@localhost ~]$ free -m
               total        used        free      shared  buff/cache   available
Mem:             457         355          27          19         106         102
Swap:            819          10         809
```
4. ajoutez votre shell `bash` actuel au *CGroup* `task1`
```
[dash@localhost ~]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task1/cgroup.procs
```
6. utilisez de nouveau `stress-ng`
```
[dash@localhost ~]$ stress-ng --vm 1 --vm-bytes 200M
stress-ng: info:  [15333] defaulting to a 1 day, 0 secs run per stressor
stress-ng: info:  [15333] dispatching hogs: 1 vm
```
8. constatez que le processus `stress-ng` est tué en boucle dès qu'il remplit la RAM au delà de la limite
```
[dash@localhost ~]$ watch -n0.1 "ps -eo cmd,pid,rss | grep stress"
```


🌞 **Créer un nouveau sous-*CGroup*** :

- appelez-le `task2`
```
[dash@localhost ~]$ sudo mkdir /sys/fs/cgroup/meow/task2
```

🌞 **Appliquer des restrictions CPU** :

- utilisez la mécanique de `cpu.weight` pour définir des priorités différentes à `task1` et `task2`
```
[dash@localhost ~]$ echo 100 | sudo tee /sys/fs/cgroup/meow/task1/cpu.weight
100
[dash@localhost ~]$ echo 500 | sudo tee /sys/fs/cgroup/meow/task2/cpu.weight
500
```
- utilisez `stress-ng` ou un bon vieux `cat /dev/random` pour lancer un processus CPU-intensive, et pour prouver que la restriction est en place
- pour tester, vous devez :
  - lancer deux shells en même temps
  - ajouter le premier au *CGroup* `task1`
Deja fait dans l'étape précédente
  - ajouter le deuxième au *CGroup* `task2`
  ```
  [dash@localhost ~]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task2/cgroup.procs
15166
  ```
  - dans les deux shells, lancer un processus CPU-intensive
```
[dash@localhost ~]$ stress-ng --cpu 1 --timeout 0
stress-ng: info:  [26503] setting to a 0 secs run per stressor
stress-ng: info:  [26503] dispatching hogs: 1 cpu
```

```
[dash@localhost ~]$ stress-ng --cpu 1 --timeout 0
stress-ng: info:  [25606] setting to a 0 secs run per stressor
stress-ng: info:  [25606] dispatching hogs: 1 cpu
```
  -  constatez avec un `htop` par exemple que les deux processus ne se répartissent pas équitablement la puissance du CPU
```
top - 00:19:14 up 13:15,  5 users,  load average: 1.21, 1.40, 1.37
Tasks: 213 total,   4 running, 209 sleeping,   0 stopped,   0 zombie
%Cpu(s): 99.1 us,  0.4 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.4 hi,  0.0 si,  0.0 st
MiB Mem :    457.9 total,    140.9 free,    163.4 used,    198.2 buff/cache
MiB Swap:    820.0 total,    758.8 free,     61.2 used.    294.5 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   25607 dash      20   0   37476   6020   3840 R  75.8   1.3   8:12.63 stress-ng-cpu
   26504 dash      20   0   37476   6080   3840 R  15.3   1.3   0:09.90 stress-ng-cpu
```
## 3. systemd

### A. One-shot

Y'a une commande rigolote et parfois pratique qui permet de jouer avec tout ça. Une commande qui permet de créer un service système temporaire à la volée en une seule ligne de commande : `systemd-run`.

> Pour plusieurs tests dans le TP, on se servira du serveur Web embarqué par Python. Il se lance en une seule commande, fait des trucs sur le système (serveur web, donc il lit des fichiers, fait des connexions réseau), c'est parfait pour faire des tests ! Pour le lancer : `python -m http.server 8888`.

🌞 **Lancer un serveur Web Python sous forme de service temporaire**

- avec la commande `python -m http.server <PORT>`
- n'oubliez pas d'ouvrir ce port dans le firewall pour tester
- il faudra le lancer avec la commande `systemd-run` pour en faire un service temporaire
- affichez le `status` du service pour prouver qu'il run
```
[dash@localhost ~]$ sudo firewall-cmd --add-port=8888/tcp --permanent
[dash@localhost ~]$ sudo firewall-cmd --reload
[dash@localhost ~]$ sudo systemd-run -u python_web python3 -m http.server 8888
Running as unit: python_web.service
[dash@localhost ~]$ sudo systemctl status python_web
● python_web.service - /bin/python3 -m http.server 8888
     Loaded: loaded (/run/systemd/transient/python_web.service; transient)
  Transient: yes
     Active: active (running) since Wed 2025-02-26 00:42:54 CET; 2min 21s ago
   Main PID: 26577 (python3)
      Tasks: 1 (limit: 2665)
     Memory: 9.1M
        CPU: 74ms
     CGroup: /system.slice/python_web.service
             └─26577 /bin/python3 -m http.server 8888

Feb 26 00:42:54 localhost.localdomain systemd[1]: Started /bin/python3 -m http.server 8888.
```

```bash
# lancer un sleep sous la forme d'un service nommé meow_test.service
sudo systemd-run -u meow_test sleep 9999
```

🌞 **Appliquer à la volée des restrictions**

- avec l'option `-p` de `systemd-run` vous pouvez préciser des paramètres pour le service
- utilisez le paramètre `MemoryMax` pour mettre en place une limite à 234M
```
[dash@localhost ~]$ sudo systemd-run -u meow_test -p MemoryMax=234M sleep 9999
Running as unit: meow_test.service
```


> En vrai, `systemd-run` est un tool vraiment pratique pour limiter l'accès aux ressources d'un process qu'on lance oneshot.

🌞 **Restrictions *CGroup* ?**

- prouvez que la restriction à 234M de `systemd-run` est mise en place avec les *CGroups* Linux
```
[dash@localhost ~]$ sudo grep -nri $(( 234 * 1024 * 1024 )) /sys/fs/cgroup 2>/dev/null
/sys/fs/cgroup/system.slice/meow_test.service/memory.max:1:245366784
```
> Les montants de RAM chelous c'pour vous permettre de faire des `grep` pour trouver facilement. Attention par contre, les restrictions automatiques appliquées par systemd sont exprimées en octets (pas KB ni MB) donc faut faire une ptite multiplication pour le trouver facilement. En l'occurence, un `grep -nri $(( 234 * 1024 * 1024 ))` depuis `/sys/fs/cgroup` fera le taff ;)

### B. Real service

🌞 **Créez un service `web.service`**

- habitués nan ? Faut créer un fichier dans `/etc/systemd/system/`
- n'oubliez pas de `sudo systemctl daemon-reload` à chaque modification
- ce service doit lancer un serveur web python sur le port 9999/tcp
```
[dash@localhost ~]$ sudo firewall-cmd --add-port=9999/tcp --permanent
[dash@localhost ~]$ sudo firewall-cmd --reload
[dash@localhost ~]$ sudo nano /etc/systemd/system/web.service
[dash@localhost ~]$ sudo systemctl daemon-reload
[dash@localhost ~]$ sudo systemctl enable web;service
[dash@localhost ~]$ sudo systemctl start web.service
[dash@localhost ~]$ sudo systemctl status web.service
● web.service - Python HTTP Server
     Loaded: loaded (/etc/systemd/system/web.service; enabled; preset: disabled)
     Active: active (running) since Wed 2025-02-26 01:35:22 CET; 6s ago
   Main PID: 27142 (python3)
      Tasks: 1 (limit: 2665)
     Memory: 8.9M
        CPU: 50ms
     CGroup: /system.slice/web.service
             └─27142 /usr/bin/python3 -m http.server 9999

```

🌞 **Restriction *CGroup***

- modifier le fichier `web.service` pour inclure des limitations mises en place par des CGroups :
  - mémoire max : 321M
  - limitation d'écriture disque : 1M
  - limitation de lecture disque : 1M
  - limitation CPU : 50% d'utilisation
```
[dash@localhost ~]$ sudo nano /etc/systemd/system/web.service
[Unit]
Description=Python HTTP Server
After=network.target

[Service]
ExecStart=/usr/bin/python3 -m http.server 9999
Restart=always
User=nobody
Group=nobody
MemoryMax=321M
IOWriteBandwidthMax=/dev/sda 1M
IOReadBandwidthMax=/dev/sda 1M
CPUQuota=50%

[Install]
WantedBy=multi-user.target


[dash@localhost ~]$ sudo systemctl daemon-reload
[dash@localhost ~]$ sudo systemctl restart web.service
[dash@localhost ~]$ sudo systemctl status web.service
● web.service - Python HTTP Server
     Loaded: loaded (/etc/systemd/system/web.service; enabled; preset: disabled)
     Active: active (running) since Wed 2025-02-26 01:42:23 CET; 5s ago
   Main PID: 27194 (python3)
      Tasks: 1 (limit: 2665)
     Memory: 8.9M (max: 321.0M available: 312.0M)
        CPU: 48ms
     CGroup: /system.slice/web.service
             └─27194 /usr/bin/python3 -m http.server 9999

Feb 26 01:42:23 localhost.localdomain systemd[1]: Started Python HTTP Server.

```

🌞 **Prouvez que ces restrictions ont été mises en place avec les *CGroups***

```
[dash@localhost ~]$ grep -nri $(( 321 * 1024 * 1024 )) /sys/fs/cgroup 2>/dev/null
/sys/fs/cgroup/system.slice/web.service/memory.max:1:336592896
[dash@localhost ~]$ grep -nri "50000" /sys/fs/cgroup 2>/dev/null
/sys/fs/cgroup/system.slice/web.service/cpu.max:1:50000 100000
[dash@localhost ~]$ sudo cat /sys/fs/cgroup/system.slice/web.service/io.max
8:0 rbps=1000000 wbps=1000000 riops=max wiops=max
```
