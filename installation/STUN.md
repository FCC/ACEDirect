# STUN Server Installation

ACE Direct uses a CentOS server to host the STUN server. The STUN server requires a service named turnserver.

Install the required dependencies:

```bash
sudo yum -y install
sudo yum install -y make gcc cc gcc-c++ wget openssl-devel libevent libevent-devel mysql-devel mysql-server
```

Install the LibEvent modules using `wget`:

```bash
wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
tar xvfz libevent-2.0.21-stable.tar.gz
cd libevent-2.0.21-stable && ./configure
sudo make && sudo make install && cd ..
```

Install the turnserver:

```bash
wget http://turnserver.open-sys.org/downloads/v3.2.3.8/turnserver-3.2.3.8.tar.gz
tar -xvzf turnserver-3.2.3.8.tar.gz
cd turnserver-3.2.3.8 && ./configure
sudo make && sudo make install
```

If desired, you may configure a turnserver config file to add user credentials and specify the listen port:

```bash
mkdir /etc/turnserver
vi /etc/turnserver/turnserver.conf
```

Edit the file and set the following parameters:

```bash
# setting static accounts
# Remember, "static" accounts are not dynamically checked by the turnserver process.
user=username:password

# listen ports
listening-port=<port>   # typically set to 3478
listening-ip=<LOCAL IP ADDRESS>
```

## Manually Starting the Service

Start the turnserver process. Use the first command to implement the config file, and the second not to:

If you want to configure the service using a file, use this syntax:

```bash
nohup turnserver -v -r  ip:port -z -c /etc/turnserver/turnserver.conf &
```

If you want to configure the service using command-line arguments, use this syntax:

```bash
nohup turnserver -L <local_IP> -v -z --min-port 10000 --max-port 20000 -n &
```

## Service Startup On Boot

The turnserver service can be configured to launch on boot. The following instructions have been validated on CentOS servers and may (or may not) work on other Linux distributions.

First create the following script and save it as /etc/rc.d/init.d/turnserver:

```bash
#!/bin/bash
# chkconfig: 2345 20 80
# description: Manage turnserver as a system service so it starts on boot

# Source function library
. /etc/init.d/functions

# This variable will be used to define what IP address turnserver listens on
# You may need to change this depending on how your network service is configured
HOST=$(hostname -I | awk '{print $1}')


# This is the port that STUN will listen on.
PORT=3478

start() {
        /usr/local/bin/turnserver -L $HOST -p $PORT -v -z --min-port 10000 --max-port 20000 -n
        echo "STUN has started successfully"
}

stop() {
        killall -9 turnserver
        echo "STUN server stopped successfully"
}

case $1 in
        start)
                start
                ;;
        stop)
                stop
                ;;
        restart)
                stop
                start
                ;;
        status)
                STATUS=$(netstat -tanp | grep turnserver | grep $PORT)
                if [ ${#STATUS} == 0 ]
                then
                        echo "STUN server is not currently running"
                else
                        echo "STUN server is currently running"
                        echo $STATUS
                fi
                ;;
        *)
                echo "Usage: service turnserver (start|stop|restart|status)"
                exit 1
                ;;
esac
exit 0
```

Make the script executable:

```bash
sudo chmod 755 /etc/rc.d/init.d/turnserver
```

Create the following systemd configuration file and save it as /usr/lib/systemd/system/turnserver.service:

```bash
[Unit]
Description=Turnserver Service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/etc/rc.d/init.d/turnserver start
;ExecStop=/root/start-turn.sh stop
;ExecReload=/root/start-turn.sh restart

[Install]
WantedBy=default.target
```

Reload the systemctl daemon:

```bash
sudo systemctl reload
```

Enable the turnserver service in systemctl:

```bash
sudo systemctl enable turnserver.service
```

Reboot the server:

```bash
sudo reboot now
```

You can use the following command to look at the ports that are in the LISTEN state.

```bash
sudo netstat -tnlp |grep turnserver
```

This should show the turnserver service listening on several ports.
