# Part II. Gotta get chrooty


## 1. Play manually

🌞 **Créez le dossier `/srv/get_chrooted/`**
```
[dash@localhost ~]$ sudo mkdir -p /srv/get_chrooted/
```

🌞 **Essayez de `chroot` à l'intérieur en lançant un shell**

- possible que ça fonctionne pas immédiatement car y'a pas de shells dans votre `chroot` :d
```
[dash@localhost ~]$ sudo chroot /srv/get_chrooted/ /bin/bash
chroot: failed to run command ‘/bin/bash’: No such file or directory
```

- déplacez le nécessaire dans `/srv/get_chrooted/` pour pouvoir lancé un shell `chroot`é à l'intérieur
```
[dash@localhost ~]$ sudo mkdir -p /srv/get_chrooted/
[sudo] password for dash: 
[dash@localhost ~]$ sudo chroot /srv/get_chrooted/ /bin/bash
chroot: failed to run command ‘/bin/bash’: No such file or directory
[dash@localhost ~]$ sudo mkdir /srv/get_chrooted/{bin,lib,lib64}
[dash@localhost ~]$ sudo cp /bin/bash /srv/get_chrooted/bin
[dash@localhost ~]$ ldd /bin/bash
	linux-vdso.so.1 (0x00007ffcc17ac000)
	libtinfo.so.6 => /lib64/libtinfo.so.6 (0x00007f69ccd1e000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f69cca00000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f69cceae000)
[dash@localhost ~]$ sudo cp /lib64/libtinfo.so.6 /srv/get_chrooted/lib64/libtinfo.so.6
[dash@localhost ~]$ sudo cp /lib64/libc.so.6 /srv/get_chrooted/lib64/libc.so.6
[dash@localhost ~]$ sudo cp /lib64/ld-linux-x86-64.so.2 /srv/get_chrooted/lib64/ld-linux-x86-64.so.2
[dash@localhost ~]$ sudo chroot /srv/get_chrooted/ /bin/bash
bash-5.1# 
```

## 2. SSH old friend

Keskivien foutr là tu vas me dire. OpenSSH, ce bro, comme d'hab, va nous faire le café.

On peut indiquer dans la conf OpenSSH qu'un nouvel utilisateur doit être automatiquement `chroot`é dans un dossier donné quand il se connecte.

🌞 **Créez un user `imsad`**

```
[dash@localhost ~]$ sudo useradd -m -d /srv/get_chrooted/home/imsad -s /bin/bash imsad
[sudo] password for dash: 
[dash@localhost ~]$ sudo passwd imsad
```
🌞 **Modifier la configuration du serveur SSH**

- dans le compte-rendu :
  - montrez une connexion SSH fonctionnelle sur le user `imsad`
  - prouvez que le `bash` ouvert est bien `chroot`é

```
[dash@localhost ~]$ sudo nano /etc/ssh/sshd_config
Match User imsad
        ChrootDirectory /srv/get_chrooted
        AllowTcpForwarding no
        X11Forwarding no
        AuthorizedKeysFile /srv/get_chrooted/home/imsad/.ssh/authorized_keys
[dash@localhost get_chrooted]$ sudo systemctl restart sshd
```  
Depuis ma machine cliente j'envoies la clé publique vers le serveur  

```
┌─[✗]─[dashboard@parrot]─[~]
└──╼ $scp ~/.ssh/id_rsa.pub dash@192.168.133.132:/tmp/
dash@192.168.133.132's password: 
id_rsa.pub                                    100%  742   670.8KB/s   00:00    
```  
De retour sur le serveur  

```
[dash@localhost get_chrooted]$ sudo mkdir -p /srv/get_chrooted/home/imsad/.ssh
[dash@localhost get_chrooted]$ sudo mv /tmp/id_rsa.pub /srv/get_chrooted/home/imsad/.ssh/authorized_keys
[dash@localhost get_chrooted]$ sudo chown -R imsad:imsad /srv/get_chrooted/home/imsad/.ssh
[dash@localhost get_chrooted]$ sudo chmod 700 /srv/get_chrooted/home/imsad/.ssh
[dash@localhost get_chrooted]$ sudo chmod 600 /srv/get_chrooted/home/imsad/.ssh/authorized_keys
```  
Finalement ça marche  

```
┌─[dashboard@parrot]─[~]
└──╼ $ssh imsad@192.168.133.132
Last login: Tue Feb 25 15:01:54 2025 from 192.168.133.128
-bash-5.1$ /bin/ls /
bin  home  lib	lib64
-bash-5.1$   
```


