---
title: "git using different ssh key on per directory basis"
date: 2022-08-28T15:45:00+02:00
draft: false
categories:
  - TIL
---

Disclaimer - I found this solution thanks to my friend [Piotr](https://github.com/bonias) who needed it for different use case, described below.

Recently I had to use multiple repositories on one server with different deploy keys. Solution [suggested by github docs](https://docs.github.com/en/developers/overview/managing-deploy-keys#using-multiple-repositories-on-one-server) was not viable, because I could not alter hostname easilly, as it suggest. It turns out, that actually you can use different keys on per directory basis, when using git. It's doable thanks to [git conditional includes](https://git-scm.com/docs/git-config#_includes):

```bash
deploy@localhost:~$ cat .gitconfig
[includeIf "gitdir:~/investment/"]
    path = .gitconfig-ip
```

```bash
deploy@localhost:~$ cat .gitconfig-ip
[core]
    sshCommand = ssh -i ~/.ssh/id_rsa_ip -F /dev/null -o IdentitiesOnly=yes
```

In sshCommand `-F /dev/null` causes `~/.ssh/config` to be ignored. `-o IdentitiesOnly=yes` instructs ssh-agent not to use default behaviour and offer any key available, but instead only the one provided via command line.

Another use case is to have different ssh keys for personal and proffessional stuff
