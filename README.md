# openvpn-initscript-slack
Init script to start, stop and show openvpn connections for Slackware Linux

## Usage
Copy the `rc.openvpn` file from here to:

    /etc/rc.d/rc.openvpn

This will all files matching name `/etc/openvpn/*conf` and start / stop / show all of them
To control each individual connection, create symlinks next to the rc.openvpn file, similar
to:

    cd /etc/rc.d
    ln -s rc.openvpn rc.openvpn@_connection-name_

where `connection-name` is also the name of a config file under /etc/openvpn, that is:

    /etc/openvpn/_connection-name_.conf
    
This is somewhat similar to using "service templates" in systems like Ubuntu. Then you can 
use the following commands to control each connection:

    /etc/rc.d/rc.openvpn@_connection-name_ start
    /etc/rc.d/rc.openvpn@_connection-name_ stop
    /etc/rc.d/rc.openvpn@_connection-name_ restart
    /etc/rc.d/rc.openvpn@_connection-name_ reload
    /etc/rc.d/rc.openvpn@_connection-name_ reconnect
    /etc/rc.d/rc.openvpn@_connection-name_ status

Without the `@connection-name` part, the commands apply to all connection files under
/etc/openvpn/ directory:

    /etc/rc.d/rc.openvpn start
    /etc/rc.d/rc.openvpn stop
    /etc/rc.d/rc.openvpn restart
    /etc/rc.d/rc.openvpn reload
    /etc/rc.d/rc.openvpn reconnect
    /etc/rc.d/rc.openvpn status

`reload` command is similar to a service restart, except a few 
options like `syslog` are not re-applied when changed in the connection configuration file. 

## Auto-starting a connection
Append the following lines to file `/etc/rc.d/rc.local`:

    if [ -x /etc/rc.d/rc.openvpn@_connection-name_ ]
	/etc/rc.d/rc.openvpn@_connection-name_ start
    fi

Append the following lines to file `/etc/rc.d/rc.local_shutdown`:

    if [ -x /etcrc.d/rc.openvpn@_connection-name_ ]
	/etc/rc.d/rc.openvpn@_connection-name_ stop
    fi


## Features
All connections are started with the following openvpn command:

    /usr/sbin/openvpn --cd "/etc/openvpn/" --config "$CONFIG_FILE" --daemon

Other options like `syslog _tunnel-name_` must be included in the given config file
(under /etc/openvpnv/ directory). Some script features include:
 * Highlighting for command output in the console
 * Support for multiple openvpn connections if needed
 * Operate on either all connections at once, or on each connection at a time
 * Include openvpn statistics for the connection in the status output
