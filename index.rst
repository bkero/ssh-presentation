
.. The Dark Arts of SSH slides file, created by
   hieroglyph-quickstart on Sun Apr 19 12:02:36 2015.


====================
The Dark Arts of SSH
====================

Where are we?
=============

.. figure:: _static/blowfish.jpg
   :align: center
   :figwidth: 65 %

   Logo © OpenSSH project, used for promotion

.. note::

   * What is SSH?
   * How is SSH used?

We're going to talk about
=========================

* Keys + Agents
* Config files
* Port forwards and tunnelling
* Libraries
* (X Forwarding)++
* Best practices

Keys
====

.. figure:: _static/keys.jpg
   :class: fill

   CC BY-SA https://flic.kr/p/nnEfaY

.. note::

   * Each is a unique identity
   * Consists of 2 parts: public and private key
   * Can have a passphrase or no passphrase

Creating Keys
=============

.. figure:: _static/keycutter.png
   :class: fill

   CC BY-SA https://flic.kr/p/dWe8Bz


Creating Keys
==============

.. code-block:: bash

   $ ssh-keygen
   Generating public/private rsa key pair.
   Enter file in which to save the key (/home/bkero/.ssh/id_rsa):
   Enter passphrase (empty for no passphrase): 
   Enter same passphrase again:
   Your identification has been saved in /home/bkero/.ssh/id_rsa.
   Your public key has been saved in /home/bkero/.ssh/id_rsa.pub.
   ...


Art!
====

.. code-block:: bash

   The key fingerprint is:
   SHA256:6Apo827Ag+KtsIU5zhCJPXeYfbrARJKH7rjdd5/Si0U bkero@localhost
   The key's randomart image is:
   +---[RSA 2048]----+
   |                 |
   |   o             |
   |  + o            |
   |.+ + + .         |
   |* + = + S E      |
   |o@ = o o .       |
   |@oB o o  ..      |
   |*Bo= o..oo..     |
   |o+=oo....o+.     |
   +----[SHA256]-----+


Private Key
===========

.. code-block:: bash
   :emphasize-lines: 2,3,5,6

   $ cat $HOME/.ssh/id_rsa
   -----BEGIN RSA PRIVATE KEY-----
   MIIEpQIBAAKCAQEAppDzIOAZlm9EsOWiu/WJy7EPt8sGsFUxukSK2WCa/1+uGoqi
   ...
   f5U2pE3Ek0txRfcw/bjjjlGDYNqDy4rUfC1FVaNaLLGgquTC/Sjpbe8=
   -----END RSA PRIVATE KEY-----

.. code-block:: bash
   :emphasize-lines: 3,4

   $ cat $HOME/.ssh/key_with_passphrase
   -----BEGIN RSA PRIVATE KEY-----
   Proc-Type: 4,ENCRYPTED
   DEK-Info: AES-128-CBC,F76824221BDE3EBF2C3D8FAE3689EC94

   TvWitQjoG2R9W3voiNIAd4/yhEs/XEnVDvVWDCqFbjxIT+KBccwSa9gYcuZBxSQZ
   ...
   -----END RSA PRIVATE KEY-----


Public Key
==========

.. code-block:: bash
   :emphasize-lines: 2

   $ cat $HOME/.ssh/id_rsa.pub
   ssh-rsa ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABA... bkero@localhost

Keys (Usage)
============

.. code-block:: bash

   $ cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

.. code-block:: bash

   $ ssh-copy-id -i $HOME/.ssh/my_new_key.pub bkero@host.example.com

.. note::

   * Placed in authorized_keys file
   * Shorcut script to do this called ssh-copy-id

Anatomy of an authorized_keys file
==================================

.. code-block:: bash

   $ cat $HOME/.ssh/authorized_keys
   ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA0m7hau2... bkero@ponderosa
   ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA3PweBtP... bkero@mozilla

.. code-block:: bash

   $ cat $HOME/.ssh/authorized_keys2
   command="/usr/games/bin/nethack -u $USER",
   no-port-forwarding,no-X11-forwarding,
   no-agent-forwarding,
   no-pty ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB... bkero@localhost

.. note::

   * Point out the command/options/key-type/key/comment
   * Ask if anybody can figure out what this is doing
   * Point out that authorized_keys2 is deprecated, added for SSH v2

Agents
======

.. figure:: _static/agent2.jpg
   :figwidth: 50 %
   :align: center

.. note::

   - Does anybody know who this is?
   - Agents are daemons that run on your computer, manage keys
   - Examples: ssh-agent, gnome-keyring, Keychain.app
   - Keys with passphrases must be unlocked to be used (can set timeout)
   - Demo (eval `ssh-agent`, env|grep SSH, logging in)

Hacky Agent Usage
=================

.. code-block:: bash

   $ eval `ssh-agent`
   Agent pid 6518 
   $ env | grep SSH
   SSH_AGENT_PID=6518
   SSH_AUTH_SOCK=/tmp/ssh-jKJJ4eWCtWjj/agent.6517

   $ ssh-add $HOME/.ssh/id_rsa
   Enter passphrase for /home/bkero/.ssh/id_rsa:
   Identity added: /home/bkero/.ssh/ponderosa (/home/bkero/.ssh/ponderosa)

Configuration
=============

.. code-block:: bash

   $ cat $HOME/.ssh/config
   Host irc
       HostName bke.ro
       UserName bkero
       Port 2228
       IdentityFile /home/bkero/.ssh/irc

   Match *.mozilla.com
       Username bkero@mozilla.com
       ForwardAgent yes # (!)


.. note::

   - Most things can be specified from the command line
   - Options can be discovered in man pages: ssh(1), ssh_config(5)
   - *CONFIG BLOCK containing hostname, port, user, identityfile*
   - *MATCH BLOCK CONTAINING *.mozilla.org*
 
Keeping your session alive
==========================
- man ssh_config(5): TCPKeepAlive
- man ssh_config(5): ServerAliveInternal

.. code-block:: bash

   $     ssh bke.ro -D1080 -N ^C
   $ autossh bke.ro -D1080 -N

- Mosh demo!

.. note::

   - TCPKeepAlive:
   - ServerAliveInterval:  

SSH Commandline
===============
* Must be preceeded by a newline
* Default keycombo is ~.

.. code-block:: bash

   $ ssh bke.ro
   (~?)
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


Tunnelling
==========
- Forward forwards

.. code-block:: bash

   $ ssh -L localhost:8080:intranet.mozilla.org:443 ssh.mozilla.com
   $ ssh -L8080:intranet.mozilla.org:443 ssh.mozilla.com

- Reverse forwards (with python2 -m SimpleHTTPServer trick)

.. code-block:: bash

   $ ssh -N -R0.0.0.0:2241:localhost:8000 bke.ro

- Dynamic forwards (SOCKS proxy)

.. code-block:: bash

   $ ssh -N -D1080 ssh.mozilla.com

- Sshuttle (transparent proxy)

.. code-block:: bash

   $ sshuttle -r ssh.mozilla.com --dns 0/0
   Connected.

X Forwarding
============
- Super secure mode:

.. code-block:: bash

   $ ssh -X bke.ro

- Oh god it's slow! Can we make faster? YES!
  
.. code-block:: bash

   $ ssh -XY bke.ro
   
- Xpra: It's like GNU Screen for X

.. code-block:: bash

   $ ssh bke.ro
   bke.ro$ xpra start :10
   bke.ro$ DISPLAY=:10 xterm
   
   $ xpra attach bkeroxpra :10

.. note::

   - You're going to want -Y. -Y disables X11 SECURITY. X is an old protocol, makes many round-trips.

Multiplexing Connections
========================

- man ssh_config(5) ControlMaster
- man ssh_config(5) ControlPath

.. code-block:: bash

   $ cat .ssh/config
   Host irc
       HostName bke.ro
       Port 2228
       ControlMaster yes
       ControlPath /home/bkero/.ssh/%h.socket

   $ time ssh irc echo
   real 0m2.549s
   
   $ time ssh irc &
   $ time ssh irc
   real 0m0.349s

Libraries
=========

* Paramiko (to embed SSH into your applications)
* Twisted Conch (to construct own servers)
* Demo (twisted app serving nyancat)

Best Practices (server)
=======================

* PermitRootLogin no
* GatewayPorts
* Keys only (authenticationmethods/
* AuthorizedKeysCommand

.. note::

   * Logging into a box as root with a password is bad practice. At least use keys.
   * Gatewayports allowsdisables users from binding to non-loopback interfaces. You probably want this enabled

That's all!
===========

- Thanks
