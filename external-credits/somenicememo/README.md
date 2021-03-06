# Credits

All this `README.md` is to credit to https://github.com/monteilhet/ssh

I kept this  file cause it's a typical _sysadmin_ cheat sheet :
* So there I find all typical flaws, so it's test data for security management system design
* And There I find quite a good overview of the common business cases, with their tasks and challenges, for an average sysadmin.

Here we almost have a technical doc for a real world infra.





Table of Contents
=================

  * [Introduction](#introduction)
  * [Installation](#installation)
  * [Configuration](#configuration)
  * [ssh keys](#ssh-keys)
     * [ssh-keygen additional options to manage known_hosts entries](#ssh-keygen-additional-options-to-manage-known_hosts-entries)
     * [Retrieve the public key from a SSH private key](#retrieve-the-public-key-from-a-ssh-private-key)
     * [Retrieve md5 fingerprint from public/private key](#retrieve-md5-fingerprint-from-public-or-private-key)
     * [convert between OpenSSH and SSH2](#convert-between-openssh-and-ssh2)
     * [Converting public key to PKCS format](#converting-public-key-to-pkcs-format)
  * [ssh agent](#ssh-agent)
     * [Using an SSH Agent](#using-an-ssh-agent)
     * [agent forwarding](#agent-forwarding)
  * [ssh client](#ssh-client)
     * [Copy the Public Key](#copy-the-public-key)
     * [ssh connection](#ssh-connection)
     * [Testing connection](#Testing-connection)
     * [Client Configuration File](#client-configuration-file)
     * [Disabling ssh host key checking](#disabling-ssh-host-key-checking)
     * [Connexion behind a proxy](#connexion-behind-a-proxy)
     * [ssh jump host](#ssh-jump-host)
     * [Local Forwarding](#local-forwarding)
     * [Remote Forwarding](#remote-forwarding)
     * [X11 Forwarding](#x11-forwarding)
  * [Tools](#tools)
     * [scp](#scp)
     * [sshfs](#sshfs)
     * [sftp](#sftp)
     * [sshpass](#sshpass)
     * [rsync](#rsync)


## Introduction

Secure Shell (better known as SSH) is a cryptographic network protocol which allows users to securely perform a number of network services over an unsecured network.


## Installation

```bash
aptitude install openssh-server
```

## Configuration


OpenSSH SSH daemon configuration file

    /etc/ssh/sshd_config

For instance some defaults values on Debian
```
PermitRootLogin without-password
X11Forwarding yes
UsePAM yes
```

**Disable root login** :
disable SSH remote login for root. This can be done by modifying the contents of a configuration file /etc/ssh/sshd_config. Look specifically for PermitRootLogin and set it to no.


 * PermitRootLogin=without-password/prohibit-password bans all interactive authentication methods, allowing only public-key

 * PermitRootLogin yes : autorise l'utilistateur root à se connecter au serveur avec un mot de passe
 * PermitRootLogin without-password : autorise l'utilistateur root à se connecter au serveur avec une clé uniquement
 * Port 22 : le port d'écoute du serveur (Port 22)
 * LoginGraceTime 120 : spécifie le laps de temps au bout duquel l'utilisateur sera déconnecté s'il ne parvient pas à s'authentifier
 * HostbasedAuthentication no : pour mettre en place une authentification basés sur les hôtes (définis dans /etc/hosts.esquiv) et non plus sur un système de clés et de mots de passe.
 * X11Forwarding yes : permet de transmettre les données d'une application graphique via la connexion ssh
 * Protocol 2 : openssh peut utiliser 2 protocoles SSH1 ou SSH2,


https://www.admin-linux.fr/nauthoriser-les-connexions-ssh-root-que-par-cle/


How to disable SSH timeout:
https://www.simplified.guide/ssh/disable-timeout



```bash
set -v
head /etc/ssh/sshd_config

# at openssh installation,  a pair of asymetric key is generated to identify the server (RSA and DSA)

ls -l /etc/ssh/ssh_host_{rsa,dsa}_key

ls -l /etc/ssh/ssh_host_{rsa,dsa}_key.pub

ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub
ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub

set +v
```
```
    head /etc/ssh/sshd_config
    #	$OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

    # This is the sshd server system-wide configuration file.  See
    # sshd_config(5) for more information.

    # This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

    # The strategy used for options in the default sshd_config shipped with
    # OpenSSH is to specify options with their default value where
    # possible, but leave them commented.  Uncommented options override the

    # at openssh installation,  a pair of asymetric key is generated to identify the server (RSA and DSA)

    ls -l /etc/ssh/ssh_host_{rsa,dsa}_key
    ls: impossible d'accéder à '/etc/ssh/ssh_host_dsa_key': Aucun fichier ou dossier de ce type
    -rw------- 1 root root 1675 mars   3 23:39 /etc/ssh/ssh_host_rsa_key

    ls -l /etc/ssh/ssh_host_{rsa,dsa}_key.pub
    ls: impossible d'accéder à '/etc/ssh/ssh_host_dsa_key.pub': Aucun fichier ou dossier de ce type
    -rw-r--r-- 1 root root 419 mars   3 23:39 /etc/ssh/ssh_host_rsa_key.pub

    ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub
    2048 SHA256:tvlEE7ZbURxvdYpeifA8SJhYViTriGMZuO1YjVM62Bc root@user-HP-Compaq-8200-Elite-CMT-PC (RSA)
    ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
    256 SHA256:NWqEewt0ezOHo2kLebF94CdYZy6oty2A+1aChAWEIB4 root@user-HP-Compaq-8200-Elite-CMT-PC (ECDSA)

    set +v
```

## ssh keys

SSH keys provide a more secure way of logging into a server with SSH than using a password alone.

### Set Up SSH Keys

Generating a key pair provides you with two long string of characters: a public and a private key.

`ssh-keygen` : authentication key generation, management and conversion
 + -b bits :  Specifies the number of bits in the key to create.  by default 2048 bits for RSA
 + -t dsa | rsa  : Specifies the type of key to create.
 + -N new_passphrase
 + -C comment
 + -f output_keyfile : Specifies the filename of the key file, by default ~/.ssh/id_rsa & ~/.ssh/id_rsa.pub
 + -q : silence ssh-keygen
 + -E fingerprint_hash : Specifies the hash algorithm used when displaying key fingerprints. “md5” “sha256”
 + -l : Show fingerprint of specified public key file (sha256 by default)
 + -y :  This option will read a private OpenSSH format file and print an OpenSSH public key to stdout.


```bash
       ssh-keygen -t rsa [-f key_filename] [-C comment] [-N ""]
```



```bash
set -v

rm -f rsa_*key*

ssh-keygen -t rsa -b 4096 -f rsa_lkey -N "" -C "test key 512b"

ssh-keygen -t rsa -f rsa_key -N "" -q -C "test key 256b"

ls -l rsa*

cat rsa_key.pub

cat rsa_lkey.pub

set +v
```

```
    rm -f rsa_*key*

    ssh-keygen -t rsa -b 4096 -f rsa_lkey -N "" -C "test key 512b"
    Generating public/private rsa key pair.
    Your identification has been saved in rsa_lkey.
    Your public key has been saved in rsa_lkey.pub.
    The key fingerprint is:
    SHA256:vBPddRpS0o8BqqY2Nn7zMFkB5Pe/8UfrrlcqKasY7e0 test key 512b
    The key's randomart image is:
    +---[RSA 4096]----+
    |       .o   oo.  |
    |       . . . oo  |
    |        . + . o+.|
    |       . + + o.+.|
    |        S o o .  |
    |       + =   .  o|
    |      B B    .ooo|
    |     + *o=. o .*.|
    |      o.o=Eo o*oo|
    +----[SHA256]-----+

    ssh-keygen -t rsa -f rsa_key -N "" -q -C "test key 256b"

    ls -l rsa*
    -rw------- 1 cnuser cnuser 1675 mars   3 15:47 rsa_key
    -rw-r--r-- 1 cnuser cnuser  395 mars   3 15:47 rsa_key.pub
    -rw------- 1 cnuser cnuser 3247 mars   3 15:47 rsa_lkey
    -rw-r--r-- 1 cnuser cnuser  739 mars   3 15:47 rsa_lkey.pub

    cat rsa_key.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7fkn23EBmNXX8KYu3ljQYJLsOJD7Uw2O9xHDSZsvAVJYjZywuZyp9hU+Y1EwHQRcsMkpkF4Qtdd/Ri1MnYEhmhyNPodANwdKWl3KyHVQVTgA35f3/WjTW/z3201xL6G0inrOtaIknH5hz1PmgC2HyerLP8qOz9pWOwUuY3o+OS+I6TKrU0N8Pq0kWhpt/dRo7ZWkKfJRRYzW9IYpYTLVBvRm35jWcnlBjyCSWeOApqQau8LF87Fqm4A5KtxyJVqt1id9t3cDdke/Pv2yL+T1Yp+UYDNbq2sehf+CtFTyIdKvkUNRAuul1n4EpGcPuermm2IOMnIBiMXTIPeJ0gyZj test key 256b

    cat rsa_lkey.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDBKd+hO5//RW1aEU/zLRsAUuCcgdOJJ0Oyeni9REr+p+rkFNiJOV6N3a19zv9ELlOhSnzF5vmyS7JD/T6frBEmv08VR5DA01qEQ553Qko6GI5Yonap0xy2NXDRM82pKa8G8UoM5VmQksJ+uRWTbdK8Cc1U9VjFNVckkEzIF0abcwRL18/FoV/MbEd7LMPyFVmZbhjmqS5pF02srlXw0f+F6uK761fweeruWZ75aK3oKjQobPOiJ0kfVafV+a4u3lMqTkV5Be1DSSkx2RJRH/K08rotN+UkFoy0P4IGbTMvV18/2hRqhNe/iSUA61Jy4J5fnu7Bw5YtEUrypjxjLucyFb8/UGhCu85CgB9Uk6nDmm+oxrHyVFCT+ofzpfGVqq/pVI923kxzC1tmbnpBuWlp9UcsLZVLDIWPAn8DPKSl1mWALN0GRrqe0O16xbiTwKag6NRJI9wL60Jybm8LX3hmK41La9iQuy0YF/NDZ2vvuXVew6IURiQOE7g8lTjLJcZWsXgOghVZkz3MyBovry8yTschw+Di/QlvyP1CeS5VKLo//E3Q8jV5b6g4O2kgSlFskJ84dkoxgunO8NYPX+9+zpTtBk1mc9QWkPDfr68pP0D/O7zSEBe8PrbUAnsoj4yY5qmiVyeT87RJBzYqFrE5tojIabfWsAtmoJUqDLTGrw== test key 512b

    set +v
```

### ssh-keygen additional options to manage known_hosts entries


```

ssh-keygen -F myhost         # shows myhosts's line in the known_hosts file
ssh-keygen -l -F myhost      # additionally shows myhost's fingerprint
ssh-keygen -R myhost         # remove myhost's line from known_hosts


```

NB Depending on ssh client configuration (HashKnownHosts yes) the hostname stored in the known_host file can be hashes.


```bash
set  -v

# search bitbucket.org host in known_hosts
ssh-keygen -F bitbucket.org

# search bitbucket.org host in known_hosts and show fingerprint
ssh-keygen -l -F bitbucket.org

# first entry in known_hosts
sed  -n 1p ~/.ssh/known_hosts

# compute hash of the first entry
ssh-keygen -l -f <(sed  -n 1p ~/.ssh/known_hosts | cut -d' ' -f2,3)

set +v
```

```
    # search bitbucket.org host in known_hosts
    ssh-keygen -F bitbucket.org
    # Host bitbucket.org found: line 1
    |1|uMSNUjUbZ05qauz/dZBLRMFkgX8=|sIB+Oczn/A1Jhg/KkpQWHYpw7lE= ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAubiN81eDcafrgMeLzaFPsw2kNvEcqTKl/VqLat/MaB33pZy0y3rJZtnqwR2qOOvbwKZYKiEO1O6VqNEBxKvJJelCq0dTXWT5pbO2gDXC6h6QDXCaHo6pOHGPUy+YBaGQRGuSusMEASYiWunYN0vCAI8QaXnWMXNMdFP3jHAJH0eDsoiGnLPBlBp4TNm6rYI74nMzgz3B9IikW4WVK+dc8KZJZWYjAuORU3jc1c/NPskD2ASinf8v3xnfXeukU0sJ5N6m5E8VLjObPEO+mN2t/FZTMZLiFqPWc/ALSqnMnnhwrNi2rbfg/rd/IpL8Le3pSBne8+seeFVBoGqzHM9yXw==

    # search bitbucket.org host in known_hosts and show fingerprint
    ssh-keygen -l -F bitbucket.org
    # Host bitbucket.org found: line 1
    bitbucket.org RSA SHA256:zzXQOXSRBEiUtuE8AikJYKwbHaxvSc0ojez9YXaGp1A

    # first entry in known_hosts
    sed  -n 1p ~/.ssh/known_hosts
    |1|uMSNUjUbZ05qauz/dZBLRMFkgX8=|sIB+Oczn/A1Jhg/KkpQWHYpw7lE= ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAubiN81eDcafrgMeLzaFPsw2kNvEcqTKl/VqLat/MaB33pZy0y3rJZtnqwR2qOOvbwKZYKiEO1O6VqNEBxKvJJelCq0dTXWT5pbO2gDXC6h6QDXCaHo6pOHGPUy+YBaGQRGuSusMEASYiWunYN0vCAI8QaXnWMXNMdFP3jHAJH0eDsoiGnLPBlBp4TNm6rYI74nMzgz3B9IikW4WVK+dc8KZJZWYjAuORU3jc1c/NPskD2ASinf8v3xnfXeukU0sJ5N6m5E8VLjObPEO+mN2t/FZTMZLiFqPWc/ALSqnMnnhwrNi2rbfg/rd/IpL8Le3pSBne8+seeFVBoGqzHM9yXw==

    # compute hash of the first entry
    ssh-keygen -l -f <(sed  -n 1p ~/.ssh/known_hosts | cut -d' ' -f2,3)
    sed  -n 1p ~/.ssh/known_hosts | cut -d' ' -f2,3
    2048 SHA256:zzXQOXSRBEiUtuE8AikJYKwbHaxvSc0ojez9YXaGp1A no comment (RSA)

    set +v
```

### Retrieve the public key from a SSH private key


```bash
set -v

# How do I retrieve the public key from a SSH private key
ssh-keygen -y -f rsa_key

set +v
```

```
    # How do I retrieve the public key from a SSH private key
    ssh-keygen -y -f rsa_key
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7fkn23EBmNXX8KYu3ljQYJLsOJD7Uw2O9xHDSZsvAVJYjZywuZyp9hU+Y1EwHQRcsMkpkF4Qtdd/Ri1MnYEhmhyNPodANwdKWl3KyHVQVTgA35f3/WjTW/z3201xL6G0inrOtaIknH5hz1PmgC2HyerLP8qOz9pWOwUuY3o+OS+I6TKrU0N8Pq0kWhpt/dRo7ZWkKfJRRYzW9IYpYTLVBvRm35jWcnlBjyCSWeOApqQau8LF87Fqm4A5KtxyJVqt1id9t3cDdke/Pv2yL+T1Yp+UYDNbq2sehf+CtFTyIdKvkUNRAuul1n4EpGcPuermm2IOMnIBiMXTIPeJ0gyZj

    set +v
```

### Retrieve md5 fingerprint from public or private key


```bash
set -v

# How do I retrieve md5 fingerprint from public key

ssh-keygen -E md5 -lf  rsa_key.pub

# or private key
ssh-keygen -E md5 -lf  rsa_key

ssh-keygen -E md5 -lf  rsa_lkey.pub

set +v
```

```
    # How do I retrieve md5 fingerprint from public key

    ssh-keygen -E md5 -lf  rsa_key.pub
    2048 MD5:eb:29:9d:34:64:c0:ac:5a:f1:93:17:3c:21:f0:02:d2 test key 256b (RSA)

    # or private key
    ssh-keygen -E md5 -lf  rsa_key
    2048 MD5:eb:29:9d:34:64:c0:ac:5a:f1:93:17:3c:21:f0:02:d2 test key 256b (RSA)

    ssh-keygen -E md5 -lf  rsa_lkey.pub
    4096 MD5:89:40:b2:8c:b6:3c:00:65:a4:12:54:77:f2:21:08:71 test key 512b (RSA)

    set +v
```


```bash
set -v

# How do I retrieve fingerprint from public/private key

ssh-keygen -lf  rsa_lkey.pub

set +v
```

```
    # How do I retrieve fingerprint from public key

    ssh-keygen -lf  rsa_lkey.pub
    4096 SHA256:vBPddRpS0o8BqqY2Nn7zMFkB5Pe/8UfrrlcqKasY7e0 test key 512b (RSA)

    set +v
```

### convert between OpenSSH and SSH2

https://burnz.wordpress.com/2007/12/14/ssh-convert-openssh-to-ssh2-and-vise-versa/


Convert OpenSSH key to SSH2 key

`-e`  : read a private or public OpenSSH key file and print to stdout the key in one of the formats specified by the -m option.  The default export format is “RFC4716”.

```bash
    ssh-keygen -e -f ~/.ssh/id_dsa.pub > ~/.ssh/id_dsa_ssh2.pub
```

Convert SSH2 key to OpenSSH key

`-i` : read an unencrypted private (or public) key file in the format specified by the -m option and print an
OpenSSH compatible private (or public) key to stdout.

```bash
    ssh-keygen -i -f ~/.ssh/id_dsa_1024_a.pub > ~/.ssh/id_dsa_1024_a_openssh.pub
```



```bash
set -v

# Convert openssh to ssh2 public key

ssh-keygen -e -f rsa_key.pub | tee rsa_key_ssh2.pub

# Convert ssh2 to openssh public key

ssh-keygen -i -f rsa_key_ssh2.pub

set +v
```

```
    # Convert openssh to ssh2 public key

    ssh-keygen -e -f rsa_key.pub | tee rsa_key_ssh2.pub
    ---- BEGIN SSH2 PUBLIC KEY ----
    Comment: "2048-bit RSA, converted by cnuser@cnuser-VirtualBox from Ope"
    AAAAB3NzaC1yc2EAAAADAQABAAABAQC7fkn23EBmNXX8KYu3ljQYJLsOJD7Uw2O9xHDSZs
    vAVJYjZywuZyp9hU+Y1EwHQRcsMkpkF4Qtdd/Ri1MnYEhmhyNPodANwdKWl3KyHVQVTgA3
    5f3/WjTW/z3201xL6G0inrOtaIknH5hz1PmgC2HyerLP8qOz9pWOwUuY3o+OS+I6TKrU0N
    8Pq0kWhpt/dRo7ZWkKfJRRYzW9IYpYTLVBvRm35jWcnlBjyCSWeOApqQau8LF87Fqm4A5K
    txyJVqt1id9t3cDdke/Pv2yL+T1Yp+UYDNbq2sehf+CtFTyIdKvkUNRAuul1n4EpGcPuer
    mm2IOMnIBiMXTIPeJ0gyZj
    ---- END SSH2 PUBLIC KEY ----

    # Convert ssh2 to openssh public key

    ssh-keygen -i -f rsa_key_ssh2.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7fkn23EBmNXX8KYu3ljQYJLsOJD7Uw2O9xHDSZsvAVJYjZywuZyp9hU+Y1EwHQRcsMkpkF4Qtdd/Ri1MnYEhmhyNPodANwdKWl3KyHVQVTgA35f3/WjTW/z3201xL6G0inrOtaIknH5hz1PmgC2HyerLP8qOz9pWOwUuY3o+OS+I6TKrU0N8Pq0kWhpt/dRo7ZWkKfJRRYzW9IYpYTLVBvRm35jWcnlBjyCSWeOApqQau8LF87Fqm4A5KtxyJVqt1id9t3cDdke/Pv2yL+T1Yp+UYDNbq2sehf+CtFTyIdKvkUNRAuul1n4EpGcPuermm2IOMnIBiMXTIPeJ0gyZj

    set +v
```

### Converting public key to PKCS format


#### Generating PKCS#1

generate PEM DER ASN.1 PKCS#1 RSA Public key  : -----BEGIN RSA PUBLIC KEY-----

```bash
ssh-keygen -f rsa_key.pub -e -m pem > rsa_key.pub.pem
```

#### Generating PKCS#8

```bash
ssh-keygen -f rsa_key.pub -e -m PKCS8 > rsa_key.pub.pem
```

## ssh agent

SSH agents can be used to hold your private SSH keys in memory. The agent will then authenticate you to any hosts that trust your SSH key.

### Using an SSH Agent

First start your agent:

```bash
    eval $(ssh-agent)
```
NB when ssh-agent is ran, it outputs some variables definition (eval allows to retrieve those variables)
```
SSH_AUTH_SOCK=/tmp/ssh-kSWOhvU7CzZB/agent.7993; export SSH_AUTH_SOCK;
SSH_AGENT_PID=7994; export SSH_AGENT_PID;
echo Agent pid 7994;
```

NB SSH agents listen on a unix socket (SSH_AUTH_SOCK).

Then add your keys to it – you’ll need to enter your passphrase for any encrypted keys:

```bash
    ssh-add ~/dir/mykey
```

List keys added in the ssh-agent

```bash
    ssh-add -L
```

`ssh-add` : adds private key identities to the authentication agent
 + -D : Deletes all identities from the agent.
 + -L : Lists public key parameters of all identities currently represented by the agent.
 + -l : Lists fingerprints of all identities currently represented by the agent.




### agent forwarding

One way to avoid copying SSH private keys around is to use the ssh-agent program on your local machine, with agent forwarding. If you SSH from your laptop to host A, and you have agent forwarding enabled, then agent forwarding allows you to SSH from host A to host B using the private key that resides on your laptop.
ssh with the -A flag, enables agent forwarding

```bash
    ssh -A myuser@myappserver.example.com
```

To allow agent forwarding for all hosts

*~/.ssh/config* :
```
Host *
   ForwardAgent yes
```


## ssh client

### Installation

```bash
aptitude install openssh-client

```


### Copy the Public Key

You can copy the public key into the new machine's authorized_keys file with the ssh-copy-id command.

`ssh-copy-id`:  use locally available keys to authorise logins on a remote machine


```
ssh-copy-id demo@198.51.100.0

cat ~/.ssh/id_rsa.pub | ssh demo@198.51.100.0 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >>  ~/.ssh/authorized_keys"
```



```bash
set -v

NS=sshdemo
sudo useradd -p $(mkpasswd "$NS") -d /home/$NS -m -g users -s /bin/bash "$NS"

# disable passwd
sudo passwd -d sshdemo
sudo passwd -S sshdemo

sudo bash -c "mkdir /home/sshdemo/.ssh && cat rsa_key.pub > /home/sshdemo/.ssh/authorized_keys"

ssh sshdemo@localhost -i rsa_key pwd

# sudo cat /home/sshdemo/.ssh/authorized_keys

sudo userdel -r $NS

set +v
```

```
    NS=sshdemo
    sudo useradd -p $(mkpasswd "$NS") -d /home/$NS -m -g users -s /bin/bash "$NS"
    mkpasswd "$NS"

    # disable passwd
    sudo passwd -d sshdemo
    passwd : expiration du mot de passe modifiée.
    sudo passwd -S sshdemo
    sshdemo NP 03/03/2018 0 99999 7 -1

    sudo bash -c "mkdir /home/sshdemo/.ssh && cat rsa_key.pub > /home/sshdemo/.ssh/authorized_keys"

    ssh sshdemo@localhost -i rsa_key pwd
    /home/sshdemo

    # sudo cat /home/sshdemo/.ssh/authorized_keys

    sudo userdel -r $NS
    userdel : l'emplacement de boîte aux lettres de sshdemo (/var/mail/sshdemo) n'a pas été trouvé

    set +v
```

### ssh connection

`ssh  [user@]hostname [command]` : OpenSSH SSH client (remote login program)
 + -A : Enables forwarding of the authentication agent connection.
 + -a : Disables forwarding of the authentication agent connection.
 + -i identity_file : file from which the identity (private key) for public key authentication
 + -p port : Port to connect to on the remote host
 + -X : Enables X11 forwarding.
 + -x : Disables X11 forwarding.

Force use of password

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no example.com
```

### Testing connection

```
ssh -T git@github.com
```



```bash
ssh -T git@github.com
```

```
    Hi monteilhet! You've successfully authenticated, but GitHub does not provide shell access.
```



### Running ssh command

```
ssh user1@server1 command

# running a local script remotely
ssh $HOST < script.sh
```

 A few ways to execute commands remotely using SSH:
 https://zaiste.net/a_few_ways_to_execute_commands_remotely_using_ssh/


### Client Configuration File

NB global client configuration file : `/etc/ssh/ssh_config`

```
Host *
#   ForwardAgent no
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   StrictHostKeyChecking ask
#   CheckHostIP yes
#   Port 22
#   Protocol 2
 HashKnownHosts yes


```
Useful option :
 * `HashKnownHosts yes` : hashes  the hostname stored in the known_host file.
 * `CheckHostIP` : If set to yes (the default), ssh will additionally check the host IP address in the known_hosts file (to detect if a host key changed due to DNS spoofing)

How to disable SSH timeout on the client side (https://www.simplified.guide/ssh/disable-timeout), use option

```
# system config in /etc/ssh/ssh_config or at user level in ~/.ssh/config
ServerAliveInterval 30
```

http://pentestmonkey.net/cheat-sheet/ssh-cheat-sheet

User-specific SSH configuration file

It is also possible to configure options for the SSH client in ~/.ssh/config.

```
Host <template>
   <option 1>
   <option 2>
```

Matching between file and cli options

| config | ssh option |
|------|------|
|   IdentityFile <file> | -i <file>|
|   Port <port>  | -p <port>|
|   ForwardX11 yes | -X  |
|   ProxyCommand <cmd> | -o ProxyCommand <cmd> |
|   StrictHostKeyChecking no  | -o StrictHostKeyChecking=no|
|   UserKnownHostsFile=/dev/null  | -o UserKnownHostsFile=/dev/null|
|   CheckHostIP no  | -o CheckHostIP=no |
| LocalForward <port local> <hôte distant>:<port distant> | -L <port local>:<hôte distant<:<port distant> |


Note that the configuration file should have a line like `Host *` followed by one or more parameter-value pairs. `Host *` means that it will match any host. Essentially, the parameters following `Host *` are the general defaults. Because the first matched value for each SSH parameter is used, you want to add the host-specific or subnet-specific parameters to the beginning of the file.

```
Host recette-*.th2.prod qt-*.th2.prod pp-*.th2.prod
 User monitor
 IdentityFile ~/.ssh/id_rsa.monitor

Host monclient-*
 IdentityFile ~/.ssh/id_rsa_clients

Host monclient-firewall
 User guest
 Port 2222
 Hostname 12.34.56.78

Host monclient-mail monclient-intranet
 User root

Host monclient-mail
 ProxyCommand ssh monclient-firewall nc 192.168.0.11 22

Host 10.0.0.1
 Port 2222
 User ptm
 ForwardX11 yes

```

### Disabling ssh host key checking

https://www.symantec.com/connect/articles/ssh-host-key-protection


Each time the SSH client connects with a server, it will store a related signature (a key) of the server. This information is stored in a file names named known_hosts. The known_hosts file itself is available in the .ssh subdirectory of the related user (on the client). In the case the signature of the server changes, SSH will protect the user by notifying about this chance.

`~/.ssh/known_hosts` file contains a convenient list of all servers to which you connect.

To reduce the risk of storing a clear picture of the network, the solution introduced was hashing the hostname. To enable this functionality, the HashKnownHosts option can be set to yes.

When you login to a remote host for the first time, the remote host's host key is most likely unknown to the SSH client. The default behavior is to ask the user to confirm the fingerprint of the host key.
```
$ ssh peter@192.168.0.100
The authenticity of host '192.168.0.100 (192.168.0.100)' can't be established.
RSA key fingerprint is 3f:1b:f4:bd:c5:aa:c1:1f:bf:4e:2e:cf:53:fa:d8:59.
Are you sure you want to continue connecting (yes/no)?
```
If your answer is yes, the SSH client continues login, and stores the host key locally in the file `~/.ssh/known_hosts`. You only need to validate the host key the first time around: in subsequent logins, you will not be prompted to confirm it again.

If your answer is yes, the SSH client continues login, and stores the host key locally in the file ~/.ssh/known_hosts. You only need to validate the host key the first time around: in subsequent logins, you will not be prompted to confirm it again.


To remove line number from known_hosts

```bash
    sed -i 3d ~/.ssh/known_hosts
```

How to get rid of : REMOTE HOST IDENTIFICATION HAS CHANGED Warning,
If a host is reinstalled and has a different key in ‘known_hosts

For instance with ansible use, host_key_checking = False

To disable (or control disabling) authentification warning, add the following lines to the beginning ssh config :

```bash
    ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no user@hostname
```

In ssh config file

```
Host 192.168.0.*
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
```

### Connexion behind a proxy


Requirement :  `apt-get install connect-proxy`

```
Host bitbucket.org
 User bba
 IdentityFile ~/.ssh/id_rsa
 ProxyCommand connect -H proxy-http:8080 %h %p
```

### ssh jump host

Historically, we are using `nc` on the jump host, to forward the connection to the target host.

```
# gateway behind proy
Host gtw
 Hostname 10.188.178.178
 User adm
 IdentityFile ~/.ssh/id_rsa
 ProxyCommand connect -S proxy:1080 %h %p

Host lab
  Hostname 192.168.50.100
  User gos
  # IdentityFile ~/.ssh/id_rsa  (not needed if use the same key)
  ProxyCommand ssh gtw "nc %h 22"
```

NB ssh jump host without nc (netcat), using `ssh -W %h:22`
https://chrigl.de/posts/2014/03/03/ssh-jump-host-without-nc-netcat.html

`-W host:port` :  Requests that standard input and output on the client be forwarded to host on port over the secure channel.  Implies -N, -T, ExitOnForwardFailure and ClearAllForwardings and works with Protocol version 2 only.


```
Host bastion
  Hostname 80.80.10.01
  User int
  IdentityFile ~/.ssh/id_rsa

Host server-10.10.*.*
  User sv
  ProxyCommand ssh -A int@bastion -W $(echo %h|cut -d- -f2):%p
```

Using command line

```bash

ssh -o UserKnownHostsFile=/dev/null \
    -o StrictHostKeyChecking=no \
    -o ProxyCommand="ssh -o UserKnownHostsFile=/dev/null \
                         -o StrictHostKeyChecking=no \
                         -i ${SSH_KEY_PATH} \
                         -W %h:%p ${DEF_USER}@${SSH_PROXY}" \
    -i ${SSH_KEY_PATH}${USER}@${SSHIP}

```


### Local Forwarding

http://pentestmonkey.net/cheat-sheet/ssh-cheat-sheet


Make services on the remote network accessible to your host via a local listener.

The service running on the remote host on TCP port 1521 is accessible by connecting to 10521 on the SSH client system

```bash
    ssh -L 127.0.0.1:10521:127.0.0.1:1521 user@10.0.0.1
```

or in ssh client config :

```
Host test
 Hostname 10.0.0.1
 LocalForward 127.0.0.1:10521 127.0.0.1:1521
```

NB can use any address of 127.0.0.0/8 subnet as local address, implicitely 127.0.0.1

```bash
    ssh -L 127.0.0.2:8080:localhost:80 root@gateway
```

* or in ssh client conf
```
LocalForward 127.0.0.2:8080 localhost:80
# http://127.0.0.2:8080 => will use port 80 on localhost
```

Using 0.0.0.0, hosts on the same network as the SSH client can also connect to the remote service

```bash
    ssh -L 0.0.0.0:10521:127.0.0.1:1521 10.0.0.1
```

In this example, 10.0.0.99 is a host that’s accessible from the SSH server.  We can access the service it’s running on TCP port 1521 by connecting to 10521 on the SSH client.
```bash
    ssh -L 127.0.0.1:10521:10.0.0.99:1521 10.0.0.1
```
Or for instance access service on port 80 on intranet connecting through firewall host, using local port 8080

```bash
    ssh -L 8080:intranet:80 root@firewall  #  http://localhost:8080
```

### Remote Forwarding

Make services on your local system / local network accessible to the remote host via a remote listener.  This sounds like an odd thing to want to do, but perhaps you want to expose a services that lets you download your tools.

The SSH server will be able to access TCP port 80 on the SSH client by connecting to 127.0.0.1:8000 on the SSH server.

```bash
    ssh -R 127.0.0.1:8000:127.0.0.1:80 10.0.0.1
```

### `X11` Forwarding


If your SSH client is also an X-Server then you can launch X-clients (e.g. Firefox) inside your SSH session and display them on your X-Server.

```bash
    SSH -X 10.0.0.1
```

* `~/.ssh/config` :

```
ForwardX11 yes
```


# Tools

## scp


scp copies files between hosts on a network.  It uses ssh for data transfer, and uses the same authentication and provides the same security as ssh.

`scp` — secure copy (remote file copy program
 + -P port : Specifies the port to connect to on the remote host.
 + -r : Recursively copy entire directories.
 + -C : Enable compression,
 + -v : Verbose mode.
 + -p : Preserves modification times, access times,


```bash
    scp [[user@]host1:]file1 ... [[user@]host2:]file
```

To upload files:

```bash
    scp /home/stacy/images/image*.jpg stacy@myhost.com:/home/stacy/archive
```

To download files:

```bash
    scp stacy@myhost.com:/home/stacy/archive/image*.jpg /home/stacy/downloads
```

To transfer files between to remote hosts

```bash
    scp someuser@alpha.com:/somedir/somefile.txt someuser@beta.com:/anotherdir
```

## `sshfs`

It is possible to use sshfs to mount a remote file system locally.

=> install sshfs package

`sshfs` -  filesystem client based on ssh

```bash
    sshfs [user@]host:[dir] mountpoint [options]
```
For instance

```bash
    sshfs 4556@sshfs.zaclys.com:/zclef /media/zclef
```
To unmount remote filesystem

```bash
    umount /media/zclef
```

## sftp


`sftp` — secure file transfer program

`sftp` is an interactive file transfer program, similar to ftp, which performs all operations over an encrypted ssh(1) transport. It may also use many features of ssh, such as public key authentication and compression.  sftp connects and logs into the specified host, then enters an interactive command mode.

```bash
    sftp -P 2222 sftp_user@89.185.10.1
```

## sshpass

`sshpass` noninteractive ssh password provider
 + `-p`password : The password is given on the command line.
 + `-f`filename : The password is the first line of the file filename.
 + `-e` : The password is taken from the environment variable "SSHPASS".





sshpass is a utility designed for running ssh using the mode referred to as "keyboard-interactive" password authentication, but in non-interactive mode.

```bash
    sshpass -f <(echo $pass) ssh -4 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no \
    -o GSSAPIAuthentication=no ${USER}@${SSHIP}
```

 + kash propagation arborescente de commande
 + pdsh boucle d'exécution de commandes à distance

## rsync

 + http://www.informatix.fr/tutoriels/php/rsync-comment-synchroniser-des-fichiers-a-travers-une-connexion-ssh-164
 + https://www.tecmint.com/rsync-local-remote-file-synchronization-commands/

```bash
`rsync` -- a fast, versatile, remote (and local) file-copying tool
  + -a, --archive : archive mode (<=> -rlptgo ) allows copying files recursively and it also preserves symbolic links, file permissions, user & group ownerships and timestamps
  + -v, --verbose : increase verbosity
  + -h : human-readable, output numbers in a human-readable format
  + -r, --recursive : recurse into directories
  + -l, --links : copy symlinks as symlinks,  When symlinks are encountered, recreate the symlink on the destination.
  + -p, --perms : preserve permissions
  + -x, --one-file-system : don't cross filesystem boundaries (=>ignore mount points)
  + -t, --times : preserve modification times
  + -g, --group : preserve group
  + -o, --owner : preserve owner (super-user only)
  + -z, --compress : compress file data during the transfer
  + -n, --dry-run : makes rsync perform a trial run that doesn’t make any changes
  + --delete : delete extraneous files from dest dirs
  + --exclude=PATTERN : exclude files matching PATTERN
  + --exclude-from=FILE : read exclude patterns from FILE
  + --max-size=SIZE : tells rsync to avoid transferring any file that is larger than the specified SIZE
  + --bwlimit=RATE : allows you to specify the maximum transfer rate for the data sent over the socket
  + --progress : show progress during transfer
  + --stats : print a verbose set of statistics on  the file transfer,

    rsync [options] source-folder copy-folder
```

Use cases:

 * Copy/Sync Files and Directory Locally

 * Copy/Sync Files and Directory to or From a Server (NB over ssh)

```bash
        # local to remote
        rsync -avz rpmpkgs/ root@192.168.0.101:/home/
        # remote to local
        rsync -avzh root@192.168.0.100:/home/tarunika/rpmpkgs /tmp/myrpms
```

NB by default rsync remote copy use ssh when specifying a target as user@host:/path/to/files, but it can be useful ot use -e ssh option to specify a specific port or private key to use, e.g.  -e 'ssh -p 2222' or -e 'ssh -i /path/to/private_key'  cf https://serverfault.com/questions/378939/do-you-need-e-ssh-for-rsync



```bash
set -v

mkdir /tmp/D

# Copy/Sync a Directory on Local Computer

# Copy/Sync a File on a Local Computer
rsync -vh /etc/passwd /tmp/D
ls -l /tmp/D

# copy directory and content
rsync -avzh /usr/share/doc/rsync /tmp/D
ls -R /tmp/D


# Copy/Sync Files and Directory to or From a Server
# by default remote rsync use (ssh)
readlink -f $(which rsh)  # check rsh symlink


rm -rf /tmp/D

set +v
```

```


    mkdir /tmp/D

    # Copy/Sync a Directory on Local Computer

    # Copy/Sync a File on a Local Computer
    rsync -vh /etc/passwd /tmp/D
    passwd

    sent 2.41K bytes  received 35 bytes  4.89K bytes/sec
    total size is 2.33K  speedup is 0.95
    ls -l /tmp/D
    total 4
    -rw-r--r-- 1 user user 2327 juil. 19 08:33 passwd

    # copy directory and content
    rsync -avzh /usr/share/doc/rsync /tmp/D
    sending incremental file list
    rsync/
    rsync/README.Debian.gz
    rsync/README.gz
    rsync/TODO.gz
    rsync/changelog.Debian.gz
    rsync/copyright
    rsync/tech_report.tex.gz
    rsync/examples/
    rsync/examples/logrotate.conf.rsync
    rsync/examples/rsyncd.conf
    rsync/scripts/
    rsync/scripts/atomic-rsync.gz
    rsync/scripts/cull_options.gz
    rsync/scripts/cvs2includes.gz
    rsync/scripts/file-attr-restore.gz
    rsync/scripts/files-to-excludes.gz
    rsync/scripts/git-set-file-times.gz
    rsync/scripts/logfilter.gz
    rsync/scripts/lsh.gz
    rsync/scripts/mnt-excl.gz
    rsync/scripts/munge-symlinks.gz
    rsync/scripts/rrsync.gz
    rsync/scripts/rsyncstats.gz

    sent 34.40K bytes  received 416 bytes  69.63K bytes/sec
    total size is 33.93K  speedup is 0.97
    ls -R /tmp/D
    /tmp/D:
    passwd  [0m[01;34mrsync[0m

    /tmp/D/rsync:
    [01;31mchangelog.Debian.gz[0m  [01;34mexamples[0m          [01;31mREADME.gz[0m  [01;31mtech_report.tex.gz[0m
    copyright            [01;31mREADME.Debian.gz[0m  [01;34mscripts[0m    [01;31mTODO.gz[0m

    /tmp/D/rsync/examples:
    logrotate.conf.rsync  rsyncd.conf

    /tmp/D/rsync/scripts:
    [01;31matomic-rsync.gz[0m  [01;31mfile-attr-restore.gz[0m   [01;31mlogfilter.gz[0m  [01;31mmunge-symlinks.gz[0m
    [01;31mcull_options.gz[0m  [01;31mfiles-to-excludes.gz[0m   [01;31mlsh.gz[0m        [01;31mrrsync.gz[0m
    [01;31mcvs2includes.gz[0m  [01;31mgit-set-file-times.gz[0m  [01;31mmnt-excl.gz[0m   [01;31mrsyncstats.gz[0m


    # Copy/Sync Files and Directory to or From a Server
    # by default remote rsync use
    readlink -f $(which rsh)
    /usr/bin/ssh


    rm -rf /tmp/D

    set +v
```
