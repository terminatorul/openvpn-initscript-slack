# openvpn-initscript-slack
Init script to start, stop and show openvpn connections for Slackware Linux

## Usage
Copy the `rc.openvpn` file from here to:

    /etc/rc.d/rc.openvpn

This will all files matching name `/etc/openvpn/*.conf`, and will start / stop / show all
of them as requested on the command line. To control each connection, create symlinks
next to the `rc.openvpn` file, similar to:
<code> cd /etc/rc.d
ln -s rc.openvpn rc.openvpn@<em>connection-name</em>
</code>
where <code><em>connection-name</em></code> is also the name of a config file under /etc/openvpn, that is:
```sh
    /etc/openvpn/__connection-name__.conf
```
This is somewhat similar to using "service templates" in systems like Ubuntu. Then you can 
use the following commands to control each connection:
```
    /etc/rc.d/rc.openvpn@__connection-name__ start
    /etc/rc.d/rc.openvpn@__connection-name__ stop
    /etc/rc.d/rc.openvpn@__connection-name__ restart
    /etc/rc.d/rc.openvpn@__connection-name__ reload
    /etc/rc.d/rc.openvpn@__connection-name__ reconnect
    /etc/rc.d/rc.openvpn@__connection-name__ status
```
Without the `@__connection-name__` part, the commands apply to all connection files under
`/etc/openvpn/` directory:
```sh
    /etc/rc.d/rc.openvpn start
    /etc/rc.d/rc.openvpn stop
    /etc/rc.d/rc.openvpn restart
    /etc/rc.d/rc.openvpn reload
    /etc/rc.d/rc.openvpn reconnect
    /etc/rc.d/rc.openvpn status
```
`reload` command is similar to a service restart, except a few 
options like `syslog` are not re-applied when changed in the connection configuration file. 

## Auto-starting a connection
Append the following lines to file `/etc/rc.d/rc.local`:
```sh
    if [ -x /etc/rc.d/rc.openvpn@__connection-name__ ]
    then
	/etc/rc.d/rc.openvpn@_connection-name_ start
    fi
```
Append the following lines to file `/etc/rc.d/rc.local_shutdown`:
```sh
    if [ -x /etcrc.d/rc.openvpn@__connection-name__ ]
    then
	/etc/rc.d/rc.openvpn@_connection-name_ stop
    fi
```

Remember to make the `rc.local_shutdown` file executable:
```sh
    chmod +x /etc/rc.d/rc.local_shutdown
```

The init script will be able see if you enabled a connection this way. But currently it
is not possible to enable a connection automatically, so this has to be done manually.

For the same purpose, you can also create systemv style links instead, under the
`/etc/rc.d/rc[-3].d/` directories and the link from `rc3.d/` will be seen by this init
script. But remember that the native method in Slackware, to enable a service, is to 
add it to the `rc.local` and `rc.local_shutdown` files.

## Features
All connections are created with the following `openvpn` command:
```sh
    /usr/sbin/openvpn --cd "/etc/openvpn/" --config "$CONFIG_FILE" --daemon
```
Other options like `syslog __tunnel-name__` must be included in the given config file
(under `/etc/openvpnv/` directory). Some script features include:
 * Highlighting for command output in the console
 * Support for multiple openvpn connections if needed
     * Operate on either all connections at once, or on each connection at a time
 * Include openvpn statistics for the connection in the status output
