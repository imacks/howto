Tunnel SSH to host via bastion
==============
You want to SSH into an _inner_ host by proxying through a _bastion_ host.

```
            +---------+      +-------+
 <you> ---> | bastion | ---> | inner |
            +---------+      +-------+
```

Tunneling is better than jump proxy because the bastion host doesn't need to have your SSH key.

```
# secret1.key is ssh key for bastion.host
# admin is username on bastion.host
# 192.168.1.100 is IP of _inner_ resolvable by bastion.host
ssh -i secret1.key -N -L 2222:192.168.1.100:22 admin@bastion.host
```

The terminal will block after successful authentication. Now anything you write to `localhost:2222` 
will be sent to `192.168.1.100:22` via _bastion_.

Open a new terminal and connect to _inner_:

```
# secret2.key is ssh key for _inner_
# admin is username on _inner_
ssh -i secret2.key -p 2222 admin@localhost
```
