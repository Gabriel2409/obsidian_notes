#todo
see [https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54](https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54)

- generate ssh key: `ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/<myname>_ed25519 -C "youremail.com"`
- create `.ssh/config` file and paste

```text
Host <myhost>.com
    User git
    Hostname github.com
    PreferredAuthentications publickey
    IdentityFile /home/user/.ssh/<myname>_ed25519
```

- test with `ssh -T git@<myhost>.com`
