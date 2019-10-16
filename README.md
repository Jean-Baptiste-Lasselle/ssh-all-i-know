# All I know about SSH

All I know about SSH :  my personal memo


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
