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

# `The complete FileSystem Permissions Setup

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


# SSH-KEYSNANNING

...
