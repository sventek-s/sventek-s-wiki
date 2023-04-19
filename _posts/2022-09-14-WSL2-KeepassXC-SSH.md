---
published: true
---

# How to use KeepassXC to serve SSH keys to WSL2 and Ubuntu

I am back on windows and I don't want to spin up linux VMs to do file editing stuff. In linux I use KeepassXC which has a windows client so I thought there
might be a solution that can allow me to use KeepassXC and WSL2 and boom found a solution.

Read more about it [here](https://code.mendhak.com/wsl2-keepassxc-ssh/) by Mendhak but he is using [Npiperelay](https://github.com/jstarks/npiperelay) and socat to do all the magic.

## Setup

`Npiperelay` allows named pipes to communicate between Linux in WSL and Windows.

```sh
# run this inside WSL2
cd ~ && wget https://github.com/jstarks/npiperelay/releases/latest/download/npiperelay_windows_amd64.zip
unzip npiperelay_windows_amd64.zip -d npiperelay && rm npiperelay_windows_amd64.zip
```

Socat allows WSL2 to communicate with `Npiperelay`.

```sh
# install socat
sudo apt install socat
```

```sh
# place this inside your .bashrc
#  Socat and npiperelay for KeepassXC SSH and WSL2

export SSH_AUTH_SOCK=$HOME/.ssh/agent.sock

ss -a | grep -q $SSH_AUTH_SOCK
if [ $? -ne 0 ]; then
    rm -f $SSH_AUTH_SOCK
    (setsid socat UNIX-LISTEN:$SSH_AUTH_SOCK,fork EXEC:"$HOME/npiperelay/npiperelay.exe -ei -s //./pipe/openssh-ssh-agent",nofork &) >/dev/null 2>&1
fi
```

Now you are done.

Do make sure to set `OpenSSH Authentication Agent` service `Start type` to `Automatic`.
Also make sure to `Enable SSH Agent intergration` and set `Use OpenSSH` in KeePassXC -> Tools -> Settings -> SSH Agent.