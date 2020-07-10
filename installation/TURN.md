# TURN Server Installation

ACE Direct uses an Ubuntu server to host the TURN server.

To install the server: `apt-get update && apt-get install coturn`

Edit: /etc/turnserver.conf and set the following variables:

```bash
listening-port = 3478
listening-ip = PRIVATE IP ADDRESS
external-ip = PUBLIC IP ADDRESS
min-port = 10000
max-port = 20000
realm = DOMAIN NAME
user = user:password
```

## TURN Server Startup
Edit `/etc/init.d/coturn` and verify the following parameters:

```bash
PATH=/usr/bin:/sbin:/usr/sbin:/bin
DESC=coturn             
NAME=coturn             
PROCNAME=turnserver                  
DAEMON=/usr/bin/turnserver
DAEMON_ARGS="-c /etc/turnserver.conf -o -v"  
PIDFILE_DIR=/var/run
PIDFILE=/var/run/$PROCNAME.pid
SCRIPTNAME=/etc/init.d/$NAME
USER=turnserver
GROUP=turnserver
```

Edit `/etc/default/coturn` and set the following parameter to have the server
start on boot:

```bash
TURNSERVER_ENABLED=1
```

## Starting the TURN Server
To start:

```bash
sudo turnserver –o –v
```
or 

```bash
sudo service start coturn
```
