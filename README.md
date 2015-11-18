


== Tunnels ==


. Expose port 80 from the remote machine to your local port 8080:

  $ ssh -L8080:localhost:80 user@example.com

  *ssh_config*: LocalForward 8080 localhost:80


. Allow hosts who can reach you (or your docker containers) to connect via your tunnel:

  $ ssh -L 0.0.0.0:8080:localhost:80 user@example.com

  *ssh_config*: LocalForward 0.0.0.0:8080 localhost:80


. Use your ssh tunnel as a (socks) proxy :

  $ ssh -D 1080 user@example.com

  *ssh_config*: DynamicForward 1080

  Pro-Tip: install [https://addons.mozilla.org/fr/firefox/addon/foxyproxy-standard/|FoxyProxy addon]


# Forward ports when you are already connected

  ~C (*after* a line return), then L8080:localhost:80, [Enter]


== Rebonds ==

  http://i.imgur.com/RMZQwxp.gif

== ssh_config tips ==


=== Basics ===

  host bstar
    user         dvador
    hostname     212.6.6.6
    port         222
    ForwardAgent Yes  # Umad?


Note: commandline options can overide ssh_config

  $ ssh bstar       # will get you with login "dvador"
  $ ssh root@bstar  # will get you with login "root"


=== keep connexions alive on long times ===

  TCPKeepAlive yes        # at TCP level
  ServerAliveInterval 25  # at application Level (inside ssh connexion)
  ServerAliveCountMax 5   # after this number of non-response, disconnect


=== Reuse existing ssh connexions ===

  *ssh_config*


