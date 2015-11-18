<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Tunnels](#tunnels)
      - [Expose port 80 from the remote machine to your local port 8080:](#expose-port-80-from-the-remote-machine-to-your-local-port-8080)
      - [Allow hosts who can reach you (or your docker containers) to connect via your tunnel:](#allow-hosts-who-can-reach-you-or-your-docker-containers-to-connect-via-your-tunnel)
      - [Use your ssh tunnel as a (socks) proxy :](#use-your-ssh-tunnel-as-a-socks-proxy-)
      - [Forward ports when you are already connected](#forward-ports-when-you-are-already-connected)
- [Bounces](#bounces)
      - [Connect to a server behind a front server](#connect-to-a-server-behind-a-front-server)
      - [Forward a port behind a ssh bastion](#forward-a-port-behind-a-ssh-bastion)
- [ssh_config tips](#ssh_config-tips)
    - [Basics](#basics)
    - [keep idle connexions alive on a long time](#keep-idle-connexions-alive-on-a-long-time)
    - [Reuse existing ssh connexions](#reuse-existing-ssh-connexions)
    - [Not ssh, but usefull:](#not-ssh-but-usefull)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Tunnels
=======


#### Expose port 80 from the remote machine to your local port 8080:

  Command line:

    ssh -L8080:localhost:80 user@example.com

  *ssh_config*:

    LocalForward 8080 localhost:80


#### Allow hosts who can reach you (or your docker containers) to connect via your tunnel:

  Command line:

    ssh -L 0.0.0.0:8080:localhost:80 user@example.com

  ssh_config:

    LocalForward 0.0.0.0:8080 localhost:80


#### Use your ssh tunnel as a (socks) proxy :

  Command line:

    ssh -D 1080 user@example.com

  ssh_config:

    DynamicForward 1080

* Pro-Tip: install [Foxyproxy addon](https://addons.mozilla.org/fr/firefox/addon/foxyproxy-standard/)


#### Forward ports when you are already connected

  ~C (*after* a line return), then `L8080:localhost:80` [Enter]

    $ [~C ]
    ssh> L9999:localhost:9999
    Forwarding port.
    $ 


Bounces
=======

![boing](http://i.imgur.com/RMZQwxp.gif)


[your workstation]-----[frontServer]----[DBserver]


#### Connect to a server behind a front server

```
  host frontserver
    hostname       server.example.com  # publicly accessible
    user           admin

  host DBserver
    hostname       192.168.2.3
    user           postgres
    proxycommand   ssh frontserver nc -q0 %h %p   # ssh executed on your workstation
                                                  # nc  executed on frontserver
                                                  # ssh to DBserver "piped" to this nc
    LocalForward   5432:localhost:5432
```

then, I can access directly to DBserver with just:
    ssh DBserver


#### Forward a port behind a ssh bastion


```
  host frontserver
    hostname       server.example.com
    user           admin
    LocalForward   8080:localhost:80
    LocalFoward    5432:192.168.2.3:5432

  host DBserver
    hostname       192.168.2.3
    user           postgres
    proxycommand   ssh frontserver nc -q0 %h %p
    LocalForward   15432:localhost:5432
```

also:

    ssh -o ProxyCommand='ssh bastion.example.com nc %h %p' user@server1.local


ssh_config tips
===============


### Basics

    host bstar
      user         dvador
      hostname     212.6.6.6
      port         222
      ForwardAgent Yes  # Umad?


Note: commandline options can overide ssh_config

    ssh bstar       # will get you with login "dvador"
    ssh root@bstar  # will get you with login "root"


### keep idle connexions alive on a long time

    TCPKeepAlive yes        # regular "ping" at TCP level
    ServerAliveInterval 25  # "ping" at application Level (inside ssh connexion, better against some firewall nazipliances)
    ServerAliveCountMax 5   # after this number of non-response, disconnect


### Reuse existing ssh connexions

  *ssh_config*


### Not ssh, but usefull:

    wrkdest:~$  nc -lp 9999 | tar xv

    wrksrc:~$ tar zcv /path/to/dir | nc wrkdest 9999



  or:
    wrksrc:~$ python -m SimplHTTPServer

    wrkdest:~$ wget http://wrksrc:8080/path/to/file
