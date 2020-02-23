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
  *  on the machine your ssh int, e.g. `export SOME_MACHINE_I_SSH_INTO=myarduinoserver1` :
```bash
# - on the machine you ssh into
ssh-keygen -Lf /etc/ssh/ssh_host_rsa_key.pub || echo "" && echo "so here is the signature of the public key: " && echo "" &&  ssh-keygen -lf /etc/ssh/ssh_host_rsa_key.pub
# You coud execute this command on any RSA public key
# file, even [~/.ssh/id_rsa.pub]
# But on [~/.ssh/id_rsa.pub] it wouldn't make sense :
# Indeed it's the host, not the client's, key, which fingerprint is computed by the ssh client, to check the identity of the machine you are connecting to with ssh protocol

```
  *  on the machine **from which** your `ssh` into `export SOME_MACHINE_I_SSH_INTO=myarduinoserver1` :

```bash
# - on the machine you ssh from
export SOME_MACHINE_HOSTNAME=myarduinoserver1
ssh-keygen -Lf /etc/ssh/ssh_host_rsa_key.pub || echo "" && echo "so here is the signature of the public key: " && echo "" &&  ssh-keygen -lf /etc/ssh/ssh_host_rsa_key.pub
# You coud execute this command on any RSA public key
# file, even [~/.ssh/id_rsa.pub]
# But on [~/.ssh/id_rsa.pub] it wouldn't make sense :
# Indeed it's the host, not the client's, key, which fingerprint is computed by the ssh client, to check the identity of the machine you are connecting to with ssh protocol

```


# Signed SSH key management

Following scenario at : https://www.vaultproject.io/docs/secrets/ssh/signed-ssh-certificates/

main logical steps :
* **step zero** setup the `certification authority` service which will _certify_ authenticity of `SSH` public keys
* **step two** (_ssh targets_ configuration) configure all machines to which I want to be able to ssh into :
  * basically, those machines have to trust the `certification authority` that was setup on **step zero**.
  * doing that is pretty easy, all you have to do is to :
    * create an empty file of path `cccc`
    * insert into that empty file, the  `cccc`

* **step three** (_ssh sources_ configuration) configure all the machine fromwhich you want to ssh into other machines.

###
