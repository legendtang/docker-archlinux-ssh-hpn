# archlinux-ssh-hpn

On Docker hub [archlinux-small-ssh-hpn](https://registry.hub.docker.com/u/yantis/archlinux-small-ssh-hpn/)
on Github [docker-archlinux-ssh-hpn](https://github.com/yantis/docker-archlinux-ssh-hpn)

[High Performance SSH/SCP - HPN-SSH](http://www.psc.edu/index.php/hpn-ssh)
OpenSSH_6.8p1-hpn14v5, OpenSSL 1.0.2a 19 Mar 2015

This small layer adds 6MB to the [archlinux-small]
(https://registry.hub.docker.com/u/yantis/archlinux-small/) layer.

This is both the server and the client. With the default being the server running as an [S6 service]
(http://skarnet.org/software/s6/index.html)

There are some nice speedup improvements to this. One of the nice features of this is the NONE cipher
for when there isn't at TTY allocated. ie: SCP. (authentication is still encrypted but your data 
isn't. So keep that in mind (ie: It is fine for images and video but not OK for your bitcoin wallet).

On the client you would use it as:
ssh user@hostname oNoneEnabled=true -oNoneSwitch=yes

You can also use one of the multithread ciphers for when a TTY is allocated like:
ssh user@hostname -oCipher=aes128-ctr


### Docker Images Structure
>[yantis/archlinux-tiny](https://github.com/yantis/docker-archlinux-tiny)
>>[yantis/archlinux-small](https://github.com/yantis/docker-archlinux-small)
>>>[yantis/archlinux-small-ssh-hpn](https://github.com/yantis/docker-archlinux-ssh-hpn)
>>>>[yantis/ssh-hpn-x](https://github.com/yantis/docker-ssh-hpn-x)
>>>>>[yantis/dynamic-video](https://github.com/yantis/docker-dynamic-video)
>>>>>>[yantis/virtualgl](https://github.com/yantis/docker-virtualgl)
>>>>>>>[yantis/wine](https://github.com/yantis/docker-wine)


## Server

The server follows these docker conventions:

* `-it` will run an interactive session that can be terminated with CTRL+C.
* `--rm` will run a temporary session that will make sure to remove the container on exit.
* `-v $HOME/.ssh/authorized_keys:/authorized_keys:ro` (optionally provide your keys authorized keys)
* `-p 49158:22` port to map to port 22.

If you have authorized public keys it will use them for both the root/docker users. I like using it
this way for example if I throw this up on AWS and I just use the same keys both both the primary
sshd server and the docker containers.

If you pass it keys it will disable password logging in. If you don't pass it authorized keys it
will enable the login via password.
The default user/password is docker/docker

```bash
docker run \
           -ti \
           --rm \
           -v $HOME/.ssh/authorized_keys:/authorized_keys:ro \
           -h docker \
           -p 49158:22 \
           yantis/archlinux-small-ssh-hpn
```


## Client

To use the client you can just run it as so to get a shell (or /bin/bash if you prefer)

```bash
docker run -ti --rm yantis/archlinux-small-ssh-hpn /bin/zsh
```

This example uploads a file to your server.

```bash
docker run -ti --rm -v ~/Downloads:/Downloads yantis/archlinux-small-ssh-hpn scp -P 49158 -oNoneEnabled=true -oNoneSwitch=yes /Downloads/alpine-3.1.3-x86_64.iso docker@monster:~/ 
```

This example uses a private key to connect to your server and runs xeyes.

```bash
xhost +si:localuser:$(whoami) >/dev/null

docker run \
           -ti \
           --rm \
           -e DISPLAY \
           -v /tmp/.X11-unix:/tmp/.X11-unix:ro \
           -v ~/.ssh/privatekey.pem:/home/docker/.ssh/privatekey.pem:ro \
           -u docker \
           yantis/archlinux-small-ssh-hpn ssh -X -i /home/docker/.ssh/privatekey.pem docker@yourserver -p 49158 -t xeyes
```

If you look at the below screenshot I ran three tests. First one is normal, second one is the NONE
cipher and the third one was normal to show no caching was going on.
The NONE cipher was twice as fast.
![](http://yantis-scripts.s3.amazonaws.com/screenshot_20150408-053726.jpg)

If you have AWS and If this is something you want to play with check out the [launch.sh](https://github.com/yantis/docker-archlinux-ssh-hpn/blob/master/launch.sh)
script on github. It will create a new AWS ec2 instance, install docker and launch the container then log you in. It should work with the AWS free tier.
