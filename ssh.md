# `ssh` notes

### Generate keys / passwordless access
Some best practices taken from [this nice
rundown](https://stribika.github.io/2015/01/04/secure-secure-shell.html) of
recommended settings.

Most portable (RSA is available on almost all servers):
```
$ cd ~/.ssh
$ ssh-keygen -t rsa -b 4096 -N ""
```

More secure, but many sys admins do not prepare servers for this
```
$ ssh-keygen -t ed25519 -N ""
```

To securely copy your key to an ssh server,
```
$ cat ~/.ssh/id_rsa.pub | ssh username@ssh.server.domain 'cat >> .ssh/authorized_keys'
```


Note: I would prefer to use the `-f filename` option.  But I've found that letting
the files go to their default location (e.g. ~/.ssh/id_rsa) allows me to not
have to call `ssh-add` afterward to register the key with the ssh-agent.  It
may be possible to use keys with custom names on remote servers I don't have
root access to, but for now I'm going with the less customized path of least
resistance.

### A simple tunnel

You can make a relatively simple tunnel to forward local network connections
through a server you have ssh access to, with ports on either end configurable.

L: Forward connections to a TCP port or Unix socket to the given server/host  
N: Optional, causes `ssh` not to execute a remote command. I like this, since
it helps me keep track of active port forwards and serves to reserve a shell
process for error and debugging messages related to the forwarded connection.  
`ssh -L [bind_address:]local_port:host_address:host_port server.domain.org -N`

I almost exclusively use this to forward connections to my machine's localhost
to that of another where a Jupyter notebook is running.  This allows me to
launch a Jupyter notebook kernel on a remote system, and then forward all my
local browser's traffic to it.  This looks like:  
```
ssh -L 127.0.0.2:8989:127.0.0.1:41375 server.domain.org -N
```

I tend to forward `127.0.0.2` or similar.  I don't know that this matters, but
`localhost` (i.e. `127.0.0.1`) is utilized by various programs and I'd rather
not potentially conflict with them somehow.

### Get the fingerprint for a host

See this [SO post](https://unix.stackexchange.com/questions/126908/get-ssh-server-key-fingerprint) for more.

If the host is your own machine:  
l: list fingerprint of given public key file  
f: specifies the file  
```
ssh-keygen -lf /etc/ssh/ssh_host_*_key.pub
```

If the host is a remote machine, you can still safely check the fingerprint
without authenticating with an untrusted system using `ssh-keyscan`.

For OpenSSH < 7.2, you can use temporary files, like  
```
$ tmpfile=$(mktemp)
$ ssh-keyscan remote.host.org > $tmpfile 2>/dev/null
$ ssh-keygen -lf $tmpfile
```

For >= 7.2, OpenSSH can read stdin, so you can do a one-liner
```
ssh-keygen -lf < (ssh-keyscan remote.host.org 2>/dev/null)
```
