Why this techlunch ?
====================

![tshirt](https://www.openbsd.org/images/tshirt-9.jpg)

We use ssh almost everyday, but I think most of us use less than 10% of what can be done.

Maybe after this lunch people will know more like 30% of OpenSSH capacities

This documentation is about OpenSSH. Some options might apply to other ssh implementations.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [General SSH tips](#general-ssh-tips)
    - [escape sequences:](#escape-sequences)
      - [force kill an ssh connection:](#force-kill-an-ssh-connection)
- [Tunnels](#tunnels)
      - [Access remote services from your workstation](#access-remote-services-from-your-workstation)
      - [Access far away remote service](#access-far-away-remote-service)
      - [Allow hosts who can reach you (or your docker containers) to connect via your tunnel:](#allow-hosts-who-can-reach-you-or-your-docker-containers-to-connect-via-your-tunnel)
    - [Remote tunnels](#remote-tunnels)
      - [Use your ssh tunnel as a (socks) proxy :](#use-your-ssh-tunnel-as-a-socks-proxy-)
      - [Forward ports when you are already connected](#forward-ports-when-you-are-already-connected)
- [ssh-agent](#ssh-agent)
- [Bounces](#bounces)
      - [Connect to a server behind a front server](#connect-to-a-server-behind-a-front-server)
      - [Forward a port behind a ssh bastion](#forward-a-port-behind-a-ssh-bastion)
- [ssh_config tips](#ssh_config-tips)
    - [Define ssh clients parameters and aliases](#define-ssh-clients-parameters-and-aliases)
    - [Keep idle connexions alive on a long time](#keep-idle-connexions-alive-on-a-long-time)
    - [Fix long delay on ssh connexion (reverse DNS issue)](#fix-long-delay-on-ssh-connexion-reverse-dns-issue)
    - [Reuse SSH Connection To Speed Up Remote Login Process](#reuse-ssh-connection-to-speed-up-remote-login-process)
- [Rock around the clo^W ssh](#rock-around-the-clo%5Ew-ssh)
    - [Transfert a lot of files across a "secure" LAN](#transfert-a-lot-of-files-across-a-secure-lan)
    - [Launch remote X commands](#launch-remote-x-commands)
    - [get a remote X11 display, help your mama](#get-a-remote-x11-display-help-your-mama)
- [TODO](#todo)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

General SSH tips
================

### escape sequences:

You can enter commands to ssh while connected, by typing the ```~``` character, after a line feed.


```
~# ~?
  Supported escape sequences:
   ~.   - terminate connection (and any multiplexed sessions)
   ~B   - send a BREAK to the remote system
   ~C   - open a command line
   ~R   - request rekey
   ~V/v - decrease/increase verbosity (LogLevel)
   ~^Z  - suspend ssh
   ~#   - list forwarded connections
   ~&   - background ssh (when waiting for connections to terminate)
   ~?   - this message
   ~~   - send the escape character by typing it twice
  (Note that escapes are only recognized immediately after newline.)
```

#### force kill an ssh connection:

![killed](https://uproxx.files.wordpress.com/2012/04/arya1.gif)


    ~.

usefull when, for example, the remote instance have just been shutdown while you where connected, and your terminal seems stuck


Tunnels
=======

When SSH connexions are established, you can also forward ports inside this connexions.

This is called Port Forwading in SSH man pages, and often SSH tunnels by people using it.

You can setup three kind of tunnels:

  - Local Forwarding

  - Remote Forwarding

  - Dynamic Forwading

![tunnels](https://media.giphy.com/media/vuGosPHIHWcSs/giphy.gif)

#### Access remote services from your workstation

  This may be the most common use of tunnels. You want to reach a service on the host you are connecting to.
  You open an local port on your workstation (here 8080) that points to the remote service (here localhost:80).

  Keep in mind that in this case, the remote service will see you connection comming from localhost, not a remote one.

  Command line:

    ssh -L8080:localhost:80 user@example.com

  *ssh_config*:

```
    host example.com
      LocalForward 8080 localhost:80


    $ ssh example.com   # implicit tunnel


```

  On an established connection:

```
user@example.com:~$ ~C 
ssh> L8080:localhost:80
Forwarding port.

user@example.com:~$
```



#### Access far away remote service



#### Allow hosts who can reach you (or your docker containers) to connect via your tunnel:

  This case is like the precedent one, plus it open your redirection to everyone
  that can access your workstation (collegues workstations, your local docker
  containers, ...)

  Be carefull about security when using this.

  The syntax is the same as the common tunnels, except you bind to "everyone" (```*```).
  By default SSH binds to ```127.0.0.1``` when ommited (see ```GatewayPorts``` in ```man ssh_config```).

  Instead of binding to ```*```, you can also bind to a specific IP address of your workstation (given by ```ifconfig``` or ```ip ad```).

  Command line:

    ssh -L *:8080:localhost:80 user@example.com

  ssh_config:

    LocalForward *:8080 localhost:80

### Remote tunnels

  Last commands show how to connect from your local area to a remote service. remote tunnel are the same, just inverted.

  You can open connection from the remote host (or its neighbours) to you local host or network.

  Command line:

    ssh -R *:8080:localhost:80 user@example.com

  ssh_config:

    RemoteForward *:8080 localhost:80

#### Use your ssh tunnel as a (socks) proxy :

  This can be very usefull if you are behind a firewall that let you connect to port 443, but filters everything on unencrypted connections like HTTP.

  This will initiate a connection from your workstation, to your server (example.com), and every software able to talk SOCKS protocol will be free on internet.

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


ssh-agent
=========

![agent](https://media.giphy.com/media/zKIRQFMPkHIdy/giphy.gif)

  ssh-agent stores keys in memory and keeps them available to ssh when you establish connection.

  It can also be "forwarded" on a remote host.

  It is usually launched automatically on your workstation when you start your x-session.

```
  ssh-add [path/to/id_rsa private key]
```

  List current keys loaded by the agent:

```
  ssh-add -l
```

  Delete all identifies:

```
  ssh-add -D

```

  Connect to server AND forward your ssh-agent:

```
  ssh -A user@example.com
```


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

### Define ssh clients parameters and aliases

  ssh_config:

```
    host deathstar
      user         dvador
      hostname     212.6.6.6
      port         222
```

Note: commandline options can overide ssh_config

```
    ssh deathstart.example.org   # Will get you with login "dvador" on port 222
    ssh root@dstar               # Will get you with login "root"
    ssh -A deathstar.example.org # Will forward your ssh-agent connexion. You should only do that if you trust deathstar server.
```



```
    host deathstar.example.org dstar
      user         dvador
      hostname     212.6.6.6
      port         222
      ForwardAgent Yes  # Umad?
```



Quizz:

  Where will you get a shell when you type ```ssh nsa.org``` and you have this ssh_config: ?

```
    host nsa.org
      hostname localhost
      user     root
```


### Keep idle connexions alive on a long time

![alive](http://i.imgur.com/AADPxtJ.jpg)

  When you let an idle ssh terminal for a long time, some router between
  your client and the server may reset the ssh tunnel because they think
  it's a dead connection. OpenSSH offers two options against this:

  - TCP keepalives
  - Server alive messages

  If they are sent, death of the connection or crash of one of the machines
  will be properly noticed.  This option only uses TCP keepalives (as opposed
  to using ssh level keepalives), so takes a long time to notice when the
  connection dies.  As such, you probably want the ServerAliveInterval option as
  well. However, this means that connections will die if the route is down
  temporarily, and some people find it annoying.

  Beware that some network appliances can detect these standard keepalive packets,
  and still consider the connection inactive and reset it.

  The server alive messages are sent through the encrypted channel and therefore
  will not be spoofable. Nor easily detectable. The TCP keepalive option enabled
  by TCPKeepAlive is spoofable.

```
    TCPKeepAlive yes        # Like a "ping" at TCP level.
    ServerAliveInterval 25  # "ping" at application Level (inside ssh connexion, better against some firewall nazipliances)
    ServerAliveCountMax 5   # after this number of non-response, disconnect
```

  also on command-line:

```
  ssh -o ServerAliveInterval=25 user@example.com
```




### Fix long delay on ssh connexion (reverse DNS issue)

  Sometimes, connecting to a server is very long. Everyting works fine, except for
  the delay. This issue is often linked to a bad reverse DNS resolution for your client IP:

  The [question has been answered on serverfault](https://serverfault.com/questions/371682/ssh-long-ssh-delay-and-resolv-conf-file).
  If you can't fix your reverse DNS, you have to disable DNS resolution done by the ssh server.

  Add the following parameter to sshd_config, and reload:

``` /etc/ssh/sshd_config
    UseDNS no
```

### Reuse SSH Connection To Speed Up Remote Login Process

    ControlMaster:
  Enables the sharing of multiple sessions over a single network connection. When
  set to “yes”, ssh(1) will listen for connections on a control socket specified
  using the ControlPath argument.  Additional sessions can connect to this socket
  using the same ControlPath with ControlMaster set to “no” (the default).  These
  sessions will try to reuse the master instance's network connection rather than
  initiating new ones, but will fall back to connecting normally if the control
  socket does not exist, or is not listening.

  *ssh_config example*

```
  host *
    ControlMaster auto
    ControlPath /dev/shm/.myusername.ssh-%r@%h:%p
```

  Just be careful that this can sometime lead to problem connection if the socket
  exist, but the Master connection is broken. In this case it may help to manually
  delete the socket specified by the ControlPath. For this reason, I advise to set
  the ControlPath to a tmpfs filesystem, so it's deleted when your machine is
  powered off.


Rock around the clo^W ssh
=========================

### Transfert a lot of files across a "secure" LAN

rsync of scp are good, but when you want to transfert _a lot_ of small files, tar is often faster. If you don't care about security, you can just spawn a tunnel with netcat commands and pipes to/from tar, like in these examples:

    wrkdest:~$  nc -lp 9999 | tar xv

    wrksrc:~$ tar zcv /path/to/dir | nc wrkdest 9999



  or:

    wrksrc:~$ python -m SimplHTTPServer

    wrkdest:~$ wget http://wrksrc:8080/path/to/file



### Launch remote X commands

    ssh -XY user@example.com xeyes


### get a remote X11 display, help your mama

![help](http://ak-hdl.buzzfed.com/static/enhanced/webdr06/2013/8/2/6/anigif_enhanced-buzz-18671-1375439971-6.gif)

You need to have a way to connect with ssh to your mama's workstation, and to get her Xauth token (you know her password, right ?)

    ssh -L5900:localhost:5900 mama@mama-laptop.example.com x11vnc

  and:

    xvncviewer localhost

  over a slow link, you will prefer these parameters:

    xtightvncviewer -encodings "tight copyrect" -quality 1 localhost   # you can even try -quality 0, if you don't need to be able to read text on remote screen


TODO
====

  - scp -3

  - sftp
  
  - sshfs

Links
=====
  - To accelerate bulk transfert with SSH, see this patch (allows to use all core for encryption routines) https://www.psc.edu/index.php/hpn-ssh
