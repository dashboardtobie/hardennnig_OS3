# Part IV : Linux Namespaces

## 1. Explore

🌞 **Utiliser /proc**

- déterminer les *namespaces* de votre `bash` actuel
```
[dash@localhost ~]$ ls -l /proc/$$/ns
total 0
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 net -> 'net:[4026531840]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 dash dash 0 Feb 26 02:28 uts -> 'uts:[4026531838]'
```  
- déterminer les *namespaces* du processuis qui porte l'identifiant PID 1
```
[dash@localhost ~]$ sudo ls -l /proc/1/ns
[sudo] password for dash: 
total 0
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 net -> 'net:[4026531840]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0 Feb 26 02:32 uts -> 'uts:[4026531838]'
```

🌞 **Lister tous les *namespaces* en cours d'utilisation**

- avec une simple commande `lsns`
```
[dash@localhost ~]$ lsns
        NS TYPE   NPROCS   PID USER COMMAND
4026531834 time        5   980 dash /usr/lib/systemd/systemd --user
4026531835 cgroup      5   980 dash /usr/lib/systemd/systemd --user
4026531836 pid         5   980 dash /usr/lib/systemd/systemd --user
4026531837 user        5   980 dash /usr/lib/systemd/systemd --user
4026531838 uts         5   980 dash /usr/lib/systemd/systemd --user
4026531839 ipc         5   980 dash /usr/lib/systemd/systemd --user
4026531840 net         5   980 dash /usr/lib/systemd/systemd --user
4026531841 mnt         5   980 dash /usr/lib/systemd/systemd --user
```

## 2. Create

### A. net

🌞 **Créer un nouveau *namespace* `network`**

- avec une commande `unshare`
- lancez un `bash` à l'intérieur
```
[dash@localhost ~]$ sudo unshare --net bash
```

🌞 **Prouvez que votre nouveau *namespace* est bien là**

- déjà un `ip a` devrait montrer aucune des cartes réseau depuis l'intérieur du *namespace*
```
[root@localhost dash]# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
- et on peut `lsns` pour voir ce nouveau *namespace*
```
[root@localhost dash]# lsns | grep net
4026531840 net       205     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026532545 net         2 27350 root   bash
```
- et on peut voir avec un `ls -al` dans `/proc` que ce nouveau terminal est dans un autre *namespace* `network`
```
[root@localhost dash]# ls -la /proc/$$/ns/net
lrwxrwxrwx. 1 root root 0 Feb 26 02:41 /proc/27350/ns/net -> 'net:[4026532545]'
```

### B. pid

🌞 **Créer un nouveau *namespace* `pid`**

- avec une commande `unshare`
- lancez un `bash` à l'intérieur
```[dash@localhost ~]$ sudo unshare --pid --fork --mount-proc bash
```

🌞 **Prouvez que votre nouveau *namespace* est bien là**

- déjà un `ps -ef` devrait pas montrer grand chose
```
[root@localhost dash]# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 03:10 pts/2    00:00:00 bash
root          20       1  0 03:10 pts/2    00:00:00 ps -ef

```
- et on peut `lsns` pour voir ce nouveau *namespace*
```
[root@localhost dash]# lsns | grep pid
4026532546 pid         2   1 root bash
```
- et on peut voir avec un `ls -al` dans `/proc` que ce nouveau terminal est dans un autre *namespace* `pid`
```
[root@localhost dash]# ls -al /proc/$$/ns/pid
lrwxrwxrwx. 1 root root 0 Feb 26 03:11 /proc/1/ns/pid -> 'pid:[4026532546]'
```

## 3. AND MY CONTAINERS

### A. Quick install

🌞 **Installer Docker sur la machine**

- suivez les instructions de la doc officielle spécifique pour notre OS !
```
[dash@localhost ~]$  sudo dnf -y install dnf-plugins-core
[dash@localhost ~]$ sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
[dash@localhost ~]$ sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
[dash@localhost ~]$ sudo usermod -aG docker $(whoami)
[dash@localhost ~]$ sudo systemctl eanble --now docker
```


### B. A simple container

🌞 **Lancer un simple conteneur qui sleep**

```
[dash@localhost ~]$ sudo docker run -d debian sleep 9999
```

🌞 **Avez les commandes de votre choix, avec le plus de détails possible, prouvez-que :**

- ce processus `sleep` est bien lancé sur votre machine, par votre utilisateur courant
```
[dash@localhost ~]$ ps -ef | grep sleep
root        3853    3833  0 20:26 ?        00:00:00 sleep 9999
dash        3931    1457  0 20:35 pts/0    00:00:00 grep --color=auto sleep
```
- ce processus `sleep` est isolé à l'aide de *namespaces*
```
## Il suffit de comparer les namespaces avec ceux de la machine hote
[dash@localhost ~]$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED          STATUS          PORTS     NAMES
3d73f5d9698e   debian    "sleep 9999"   58 minutes ago   Up 58 minutes             trusting_sutherland
[dash@localhost ~]$ sudo docker inspect -f '{{.State.Pid}}' trusting_sutherland
3853
[dash@localhost ~]$ sudo readlink /proc/1/ns/pid
pid:[4026531836]
[dash@localhost ~]$ sudo readlink /proc/3853/ns/pid
pid:[4026532549]
[dash@localhost ~]$ sudo readlink /proc/1/ns/net
net:[4026531840]
[dash@localhost ~]$ sudo readlink /proc/3853/ns/net
net:[4026532551]
```
- un *CGroup* a été attribué à ce conteneur
```
[dash@localhost ~]$ sudo cat /proc/3853/cgroup
0::/system.slice/docker-3d73f5d9698e8d49816dfbdebc55553aad09f37daf947c7a4547fd82c0a894a3.scope
```
- tout nouveau processus "dans" le conteneur est lui aussi isolé (voir ci-dessous pour lancer d'autres process dans le conteneur)
```
[dash@localhost ~]$ sudo docker exec -it trusting_sutherland bash
root@3d73f5d9698e:/# apt update -y
root@3d73f5d9698e:/# apt install -y procps iproute2 iputils-ping
root@3d73f5d9698e:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 19:26 ?        00:00:00 sleep 9999
root          18       0  0 20:34 pts/0    00:00:00 bash
root         453      18  0 20:38 pts/0    00:00:00 ps -ef
root@3d73f5d9698e:/# ip a            
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 72:5d:49:96:25:5d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
root@3d73f5d9698e:/# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 19:26 ?        00:00:00 sleep 9999
root          18       0  0 20:34 pts/0    00:00:00 bash
root         460      18  0 21:03 pts/0    00:00:00 ps -ef
```

### C. CGroup

Et les CGroups dans tout ça ?

🌞 **Lancer un conteneur restreint**

```
[dash@localhost ~]$ sudo docker run -d --memory=456m --name limited_container debian sleep 9999
```

🌞 **CGroup ?**

- prouvez que cette limite a été mise en place avec une conf CGroup
```
[dash@localhost ~]$ sudo docker inspect -f '{{.State.Pid}}' limited_container
4803
[dash@localhost ~]$ sudo cat /proc/4803/cgroup
0::/system.slice/docker-6fac10e55d6837c8ad8cae9f6d69ff0b0d23e409c8dd06ef9899ae3b835ef5db.scope
[dash@localhost ~]$ sudo cat /sys/fs/cgroup/system.slice/docker-6fac10e55d6837c8ad8cae9f6d69ff0b0d23e409c8dd06ef9899ae3b835ef5db.scope/memory.max
478150656

```

