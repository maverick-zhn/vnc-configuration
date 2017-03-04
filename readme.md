#Configuring VNC Server on Oracle Linux [1,2]

## 1.) Install the **tigervnc-server**

```
sudo yum install tigervnc-server
```

## 2.) Create the VNC environment for the VNC users
```
# su - maverick
$ vncpasswd
Password: password
Verify: password
```

## 3.) Display enviromental variable [4]
```
echo $DISPLAY
```

## 4.) Modifiy the configuration file 

### 4.1) Copy configuration file template
```
# cp /lib/systemd/system/vncserver@.service /lib/systemd/system/vncserver@:0.service
```

### 4.2) Edit configuration file
Replace all instances of **<USER>** with the username that will run the VNC desktop
```
ExecStart=/sbin/runuser -l maverick -c "/usr/bin/vncserver %i"
PIDFile=/home/vncuser/.vnc/%H%i.pid
```

## 5.) Reload daemon
```
# systemctl daemon-reload
```

## 6.) Enable Service
```
# systemctl enable vncserver@:0.service
```

## 7.) Start Service
```
# systemctl start vncserver@:0.service
```

# VNC Clients

```
# yum install tigervnc
```

## Connect to VNC Server
```
# vncviewer machine-name:port
```

### Suggested test suite
* Test with localhost:port
* Analyze open ports
```
# netstat -an | grep 'LISTEN'
```
* If VNC Server application ports are open, then try from VNC client.

# Troubleshouting
I have found some problems while configuring the VNC-Server on Oracle Linux. In addition, I have included all the fixes for those.

## Disabled Ethernet Device
 Manually enable Ethernet Device [3]
 ```
 sudo ifup <ethernet device>
 ```

## I couldn't connect from VNC-**client** [Oracle Linux Client-Node]
* Use ICMP to determine if VNC-Server is reachable [ping vnc-server].
* In the case that vnc-server is up, scan open ports in the server:
```
sudo nmap -n -PN -sT -sU -p- vnc-server-ip-address
```
* Look for the VNC-Server specific port. If **filtered** you must add one rule in the Oracle Linux (vnc-server) firewall (ipchains or iptables). Please refer to **firewall commands** section to open ports in the firewall.
* 

### Firewall commands
Display available zones
```
# firewall-cmd --get-zones
```

Disabling firewall
```
# service ipchains stop
```
or
```
# service iptables stop
```

Opening ports [2]
```
# firewall-cmd --zone=zone --add-service=vnc-server
# firewall-cmd --zone=zone --add-service=vnc-server --permanent
# firewall-cmd --zone=zone --add-port=<PORT_NUMBER>/tcp
# firewall-cmd --zone=zone --add-port=<PORT_NUMBER>/tcp --permanent
```

Example
```
sudo firewall-cmd --get-zones
```
Results
```
work drop internal external trusted home dmz public block
```

Adding rules to the firewall
```
[maverick@localhost .vnc]$ sudo firewall-cmd --zone=trusted --add-service=vnc-server
success
[maverick@localhost .vnc]$ sudo firewall-cmd --zone=trusted --add-service=vnc-server --permanent
[maverick@localhost .vnc]$ sudo firewall-cmd --zone=trusted --add-port=5904/tcp
success
[maverick@localhost .vnc]$ sudo firewall-cmd --zone=trusted --add-port=5904/tcp --permanent

```


# References
* [1] [Configureing VNC Server](https://oracle-base.com/articles/linux/configuring-vnc-server-on-linux)
* [2] [VNC config](https://docs.oracle.com/cd/E52668_01/E54669/html/ol7-vnc-config.html)
* [3] [Oracle Network Configuration](https://docs.oracle.com/cd/E52668_01/E54669/html/ol7-s6-netconf.html)
* [4] [Linux display Variable](http://gerardnico.com/wiki/linux/display)
