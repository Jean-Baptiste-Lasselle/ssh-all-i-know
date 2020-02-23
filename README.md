# All I know about SSH

All I know about SSH :  my personal memo

# Resume this memo edition

```bash
export WORK_FOLDER=~/ssh-all-i-know.dev
export SSH_URI_TO_THIS_REPO=git@github.com:Jean-Baptiste-Lasselle/ssh-all-i-know.git

export COMMIT_MESSAGE=""
export COMMIT_MESSAGE="$COMMIT_MESSAGE Reprise du travail sur [$SSH_URI_TO_THIS_REPO]"

initializeIAAC $SSH_URI_TO_THIS_REPO $WORK_FOLDER

atom .
# git add --all && git commit -m "$COMMIT_MESSAGE" && git push -u origin master

```

* (if u dont have th iaac bash plugin installed, and the `initializeIAAC` command) :

```bash
export WORK_FOLDER=~/ssh-all-i-know.dev
export SSH_URI_TO_THIS_REPO=git@github.com:Jean-Baptiste-Lasselle/ssh-all-i-know.git

export COMMIT_MESSAGE=""
export COMMIT_MESSAGE="$COMMIT_MESSAGE Reprise du travail sur [$SSH_URI_TO_THIS_REPO]"

git clone $SSH_URI_TO_THIS_REPO $WORK_FOLDER

cd $WORK_FOLDER
git status

atom .
# git add --all && git commit -m "$COMMIT_MESSAGE" && git push -u origin master

```

# The most simple, standard Key authentication setup (**unfit** for any production)

* on the machine(s) you want to ssh from :


```bash

export WHERE_TO_CREATE_RSA_KEY_PAIR=${WHERE_TO_CREATE_RSA_KEY_PAIR:'~/.ssh'}
# just options, see end of script
export ROBOTS_ID=cerno-alpha
export PRIVATE_KEY_FULLPATH=$WHERE_TO_CREATE_RSA_KEY_PAIR/myrobot-${ROBOTS_ID}-key_rsa
export PRIVATE_KEY_FULLPATH=$WHERE_TO_CREATE_RSA_KEY_PAIR/id_rsa

# --- #
# Ask hashicorp vault to generate a passphrase for you :
# https://github.com/sethvargo/vault-secrets-gen
# Yes, this morning I'm gonna change my passpword!
#
export DEFAULT_PRIVATE_KEY_PASSPHRASE="Etre ou ne pas etre, telle est la question"
# - p***** mais ouiii pour gitlab.com, la clef enregistrée ne DOIT PAS avoir de passphrase, sinon l'authentification foire !!!
# ou alors il faut tester comment faire la passphrase en mode command line silenceieux
export DEFAULT_PRIVATE_KEY_PASSPHRASE=""
# p***** ouais c'est énorme j'ai bien testé que la [passphrase] fait échouer l'auth. [gitlab.com]
#
export LE_COMMENTAIRE_DE_CLEF="[$ROBOTS_ID]-robotsrocks@$(hostname)"

ssh-keygen -C $LE_COMMENTAIRE_DE_CLEF -t rsa -b 4096 -f $PRIVATE_KEY_FULLPATH -q -P "$DEFAULT_PRIVATE_KEY_PASSPHRASE"
# --
# If you're a little more comfortable with ssh, you can set a custom path for the generated files, suing [-f] option.
# Indeed, when you're out of defaults, ssh clients might get a little harsh (and that's a good, and meant a thing)
# So do that if you're a bit experienced with system administration and ssh clients/servers.
# --
# ssh-keygen -C $LE_COMMENTAIRE_DE_CLEF -t rsa -b 4096 -f $PRIVATE_KEY_FULLPATH -q -P "$DEFAULT_PRIVATE_KEY_PASSPHRASE"

ls -allh $WHERE_TO_CREATE_RSA_KEY_PAIR

echo "now you can ssh-connect to machines using : "
echo "ssh -i ${PRIVATE_KEY_FULLPATH} <user>@<hostname>"

```

* On the machine you want to ssh into :

```bash
# -
sudo apt-get update -y

sudo apt-get install -y openssh-server

# --
# Inside the [/etc/ssh/sshd_config] conf file :
#
# 1./ uncomment, and leave value set to 'yes', the
# '#PubkeyAuthentication yes'
# for the 'PubkeyAuthentication' configuration parameter
#
# 2./ uncomment, and set value to 'no', the
# '#PasswordAuthentication yes'   => 'PasswordAuthentication no'
# for the 'PubkeyAuthentication' configuration parameter
#
# --
# After it's done, restart openssh server using :
# sudo systemctl restart sshd.service
# --
# Good practices : use hashicorp packer to pxeless boot
# --
```



# The complete FileSystem Permissions Setup

(credits to https://gist.github.com/grenade/6318301 )

* `generate-ssh-key.sh` :

```bash
ssh-keygen -t rsa -b 4096 -N '' -C "jblasselle@gopeg.io" -f ~/.ssh/id_rsa
ssh-keygen -t rsa -b 4096 -N '' -C "jblasselle@gopeg.io" -f ~/.ssh/github_rsa
ssh-keygen -t rsa -b 4096 -N '' -C "jblasselle@gopeg.io" -f ~/.ssh/mozilla_rsa
```

* `ssh-key-add.sh` :

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/github_rsa
ssh-add ~/.ssh/mozilla_rsa
```

* `ssh-key-permissions.sh` :

```bash
chmod 700 ~/.ssh
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/known_hosts
chmod 644 ~/.ssh/config
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/github_rsa
chmod 644 ~/.ssh/github_rsa.pub
chmod 600 ~/.ssh/mozilla_rsa
chmod 644 ~/.ssh/mozilla_rsa.pub
```

# SSH-KEYSNANNING, SSH hosts fingerprints

* on an `*nix` system on which it is installed, and running, the ssh server identifies itself using a public RSA key (which has a paired private rsa key), located at a specific path on the system, `/etc/ssh/ssh_host_rsa_key.pub`
* Hence, if I want to ssh  into `export SOME_MACHINE_HOSTNAME=myarduinoserver1`, from `export MY_USUAL_WORKSTATION=pc_asus_16gbram4corescpu` :
  * on `SOME_MACHINE_HOSTNAME`, there is a public RSA key at `/etc/ssh/ssh_host_rsa_key.pub`
  * when, from  `MY_USUAL_WORKSTATION` , you try and ssh into `SOME_MACHINE_HOSTNAME`, the ssh client on `MY_USUAL_WORKSTATION` grabs the public key at `/etc/ssh/ssh_host_rsa_key.pub` on `SOME_MACHINE_HOSTNAME`, and computes its fingerprint, using either the `ssh-keygen -lf `, or the `ssh-keygen -Lf ` command.  This fingerprint is compared to another fingerprint : computed by the ssh client, using the `~/.ssh/known_hosts` to check the identity of the machine you are connecting to with `ssh` protocol
* when you ssh into `export SOME_MACHINE_HOSTNAME=myarduinoserver1`, the openssh server
  *  on the machine you ssh into, e.g. `export SOME_MACHINE_I_SSH_INTO=myarduinoserver1` :
```bash
# - on the machine you ssh into
ssh-keygen -Lf /etc/ssh/ssh_host_rsa_key.pub || echo "" && echo "so here is the signature of the public key: " && echo "" &&  ssh-keygen -lf /etc/ssh/ssh_host_rsa_key.pub
# You coud execute this command on any RSA public key
# file, even [~/.ssh/id_rsa.pub]
# But on [~/.ssh/id_rsa.pub] it wouldn't make sense :
# Indeed it's the host, not the client's, key, which fingerprint is computed by the ssh client, to check the identity of the machine you are connecting to with ssh protocol

```
  *  on the machine **from which** you `ssh` into `export SOME_MACHINE_I_SSH_INTO=myarduinoserver1` :

```bash
# - on the machine you ssh from
export SOME_MACHINE_HOSTNAME=myarduinoserver1

ssh-keygen -Lf /etc/ssh/ssh_host_rsa_key.pub || echo "" && echo "so here is the signature of the public key: " && echo "" &&  ssh-keygen -lf /etc/ssh/ssh_host_rsa_key.pub
# You coud execute this command on any RSA public key
# file, even [~/.ssh/id_rsa.pub]
# But on [~/.ssh/id_rsa.pub] it wouldn't make sense :
# Indeed it's the host, not the client's, key, which fingerprint is computed by the ssh client, to check the identity of the machine you are connecting to with ssh protocol

```

### SSH Keyscanning

```bash
rm -f ~/.ssh/known_hosts
ssh-keyscan github.com >> ~/.ssh/known_hosts
ssh-keyscan gitlab.com >> ~/.ssh/known_hosts

cat ~/.ssh/known_hosts

echo "Provided you registered your public SSH RSA key for your gitlab.com's and github.com's user accounts"
ssh -Tai ~/.ssh/id_rsa git@gitlab.com
ssh -Tai ~/.ssh/id_rsa git@github.com


```
* example output :

```
jbl@poste-devops-jbl-16gbram:~$ ssh-keyscan gitlab.com
# gitlab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
gitlab.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAfuCHKVTjquxvt6CM6tdG4SLp1Btn/nOeHHE5UOzRdf
# gitlab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
gitlab.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCsj2bNKTBSpIYDEGk9KxsGh3mySTRgMtXL583qmBpzeQ+jqCMRgBqB98u3z++J1sKlXHWfM9dyhSevkMwSbhoR8XIq/U0tCNyokEi/ueaBMCvbcTHhO7FcwzY92WK4Yt0aGROY5qX2UKSeOvuP4D6TPqKF1onrSzH9bx9XUf2lEdWT/ia1NEKjunUqu1xOB/StKDHMoX4/OKyIzuS0q/T1zOATthvasJFoPrAjkohTyaDUz2LN5JoH839hViyEG82yB+MjcFV5MU3N1l1QL3cVUCh93xSaua1N85qivl+siMkPGbO5xR/En4iEY6K2XPASUEMaieWVNTRCtJ4S8H+9
# gitlab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
gitlab.com ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFSMqzJeV9rUzU4kWitGjeR4PWSa29SPqJ1fVkhtj3Hw9xjLVXVYrU9QlYWrOLXBpQ6KWjbjTDTdDkoohFzgbEY=
jbl@poste-devops-jbl-16gbram:~$ ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
# gitlab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
# gitlab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
# gitlab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
jbl@poste-devops-jbl-16gbram:~$ ssh-keyscan github.com >> ~/.ssh/known_hosts
# github.com:22 SSH-2.0-babeld-17f81433
# github.com:22 SSH-2.0-babeld-17f81433
# github.com:22 SSH-2.0-babeld-17f81433

jbl@poste-devops-jbl-16gbram:~$ ssh -Tai ~/.ssh/id_rsa git@gitlab.com
Welcome to GitLab, @pegasus.devops!
jbl@poste-devops-jbl-16gbram:~$ ssh -Tai ~/.ssh/id_rsa git@github.com
Hi Jean-Baptiste-Lasselle! You've successfully authenticated, but GitHub does not provide shell access.
jbl@poste-devops-jbl-16gbram:~$
```


* checking fingerprint :
  * create a file `test.sh`, and add the following content into it  :
```bash
#!/bin/bash

echo ''
echo ''
echo ' +++>> This is a simple shell script'
echo " All that script does is executing on [$(hostname)] : "
echo ''
echo "ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub"
echo "ssh-keygen -l -f ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub"
echo ''
echo "Ok so now the RSA fingerprint of the ssh host on [$(hostname)] is : "
echo ''
set -x
ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub
set +x

echo ''
echo "Ok so now the ECDSA fingerprint of the ssh host on [$(hostname)] is : "
echo ''

set -x
ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
set +x

echo ''
echo "Ok so now the ED25519 fingerprint of the ssh host on [$(hostname)] is : "
echo ''

set -x
ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub
set +x

echo "list of [/etc/ssh/ssh_host_*] files : "
ls -allh /etc/ssh/ssh_host_*

```
  * execute the following :
```bash
export GRABBED_REMOTE_MACHINES_SSH_HOST_PUB_KEY=$(mktemp)
export HOSTNAME_TO_TEST=pegasusio.io
ssh-keyscan $HOSTNAME_TO_TEST > $GRABBED_REMOTE_MACHINES_SSH_HOST_PUB_KEY 2> /dev/null
echo ''
echo "First, on the client side : we can grab [${HOSTNAME_TO_TEST}]'s SSH host public RSA key, and calculate its [fingerprint], like this : "
echo ''

ssh-keygen -l -f $GRABBED_REMOTE_MACHINES_SSH_HOST_PUB_KEY
echo ''

echo "Press any key to proceed"
# read waithere1
echo ''

echo "Then, on the server side : we can ssh connect into [${HOSTNAME_TO_TEST}], and run [ssh-keygen -l -f ] directly on the content of the file [/etc/ssh/ssh_host_rsa_key.pub]  "
echo ''
ssh -i ~/.ssh/id_rsa $(whoami)@${HOSTNAME_TO_TEST} < test.sh

rm $GRABBED_REMOTE_MACHINES_SSH_HOST_PUB_KEY

```


* example output :

```
jbl@poste-devops-jbl-16gbram:~$ ssh -i ~/.ssh/id_rsa $(whoami)@${HOSTNAME_TO_TEST} < test.sh
Pseudo-terminal will not be allocated because stdin is not a terminal.
Linux poste-devops-typique 4.9.0-12-amd64 #1 SMP Debian 4.9.210-1 (2020-01-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.


 +++>> This is a simple shell script
 All that script does is executing on [poste-devops-typique] :

ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub
ssh-keygen -l -f ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub

Ok so now the RSA fingerprint of the ssh host on [poste-devops-typique] is :

+ ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key.pub
2048 SHA256:5RTaTSYFV5OjIGmIrGhxm+F3IqUCcVP7pnR/E8YNx0w root@poste-devops-typique (RSA)

+ set +x
Ok so now the ECDSA fingerprint of the ssh host on [poste-devops-typique] is :

+ ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
256 SHA256:2PVf9IRs3wPWV5Ch9m2yZM3DmRoF4UMwBoE1oB7evf8 root@poste-devops-typique (ECDSA)
+ set +x

Ok so now the ED25519 fingerprint of the ssh host on [poste-devops-typique] is :

+ ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub
256 SHA256:enNfv4K8BFESAU0ohF9L5UfEGozUWRZliK+hkMSzJUs root@poste-devops-typique (ED25519)
+ set +x
list of [/etc/ssh/ssh_host_*] files :
-rw------- 1 root root  227 févr. 22 20:30 /etc/ssh/ssh_host_ecdsa_key
-rw-r--r-- 1 root root  187 févr. 22 20:30 /etc/ssh/ssh_host_ecdsa_key.pub
-rw------- 1 root root  419 févr. 22 20:30 /etc/ssh/ssh_host_ed25519_key
-rw-r--r-- 1 root root  107 févr. 22 20:30 /etc/ssh/ssh_host_ed25519_key.pub
-rw------- 1 root root 1,7K févr. 22 20:30 /etc/ssh/ssh_host_rsa_key
-rw-r--r-- 1 root root  407 févr. 22 20:30 /etc/ssh/ssh_host_rsa_key.pub
jbl@poste-devops-jbl-16gbram:~$
```


# Signed SSH key management with HashiCorp Vault

Following scenario at : https://www.vaultproject.io/docs/secrets/ssh/signed-ssh-certificates/

What the SSH clients use to everyday use their SSHkye that can expire : https://github.com/hashicorp/vault-ssh-helper

main logical steps :
* **step zero** setup the `certification authority` service which will _certify_ authenticity of `SSH` public keys
* **step two** (_ssh targets_ configuration) configure all machines to which I want to be able to ssh into :
  * basically, those machines have to trust the `certification authority` that was setup on **step zero**.
  * doing that is pretty easy, all you have to do is to :
    * create an empty file of path `cccc`
    * insert into that empty file, the  `cccc`

* **step three** (_ssh sources_ configuration) configure all the machine fromwhich you want to ssh into other machines.

###


# The SSH Bastion, and HashiCorp Vault expirable SSH Key management


### Towards a production setup

* Existing solutions / standards in th scope of ssh bastions, reagarless of any specific SSH Key management policy in place in a given organization, or `HashiCorp Vault`.
* Main business cases, still regardless of any SSH key  management policy choice, or `HashiCorp Vault`.
* Integration with HashiCorp Vault SSH Key management.


### A few compressed notes on teleport

Trying a setup with `teleport` :

https://github.com/gravitational/teleport

Looks great n sexy, love the feature list, and business cases described.


* I'll focus on the  SSH Bastion feature of teleport, with :
  * standard bastion ssh conection tests
  * **SSH-over-HTTPS** : mandatory,if not most important test to setup, goal is to reach `DCIM` solution
* I'll focus on the on autdit features on ssh access and kubectl history,
* I'll add a monitoring solution based on :
  * `Elastic Beats` + `Prometheus exporters ` as data collector, logged based and time series based
  *  monitoring data displayed with :
    * one `Grafana` Dashboard _`SSH` Access Control Audit_
    * one `Grafana` Dashboard _`kubectl` Access Control Audit_
* To finish with, I'll add `teleport`'s extras features : sftp
