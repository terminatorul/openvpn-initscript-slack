# OpenVPN connections as system services in Slackware Linux
Slackware Linux has no specific init script for openvpn and openvpn connections.
Slackware documentation for openvpn suggests to copy/paste the content of a
minimal init script to the appropriate system file.

Here is a more useful init script to start, stop and show openvpn connections
for Slackware Linux.

## Usage
Copy the `rc.openvpn` file from here to:
```
    /etc/rc.d/rc.openvpn
```
This will search for all connection files matching the name `/etc/openvpn/*.conf`, and will start, stop or show all
of them as requested on the command line. To control each connection, create symlinks
next to the `rc.openvpn` file, similar to:
<pre><code>    cd /etc/rc.d
    ln -s rc.openvpn rc.openvpn@<em>connection-name</em></code></pre>
where <code><em>connection-name</em></code> is also the name of a config file under `/etc/openvpn/`, that is:
<pre><code>    /etc/openvpn/<em>connection-name</em>.conf</code></pre>
If you want o keep connections in separate sub-directories under `/etc/openvpn/` (to separate the keys,
certificates and ipp files), you can still do that and create a symlink in the expected place to the
.ovpn file:
<pre><code>    cd /etc/openvpn/
    ln -s <em>connection-name</em>/<em>connection-name</em>.ovpn <em>connection-name</em>.conf</code></pre>
The <code>@<em>connection-name</em></code> syntax is similar to using "service templates" in systems like
Ubuntu. Then you can use the following commands to control each connection:
<pre><code>    /etc/rc.d/rc.openvpn@<em>connection-name</em> start
    /etc/rc.d/rc.openvpn@<em>connection-name</em> stop
    /etc/rc.d/rc.openvpn@<em>connection-name</em> restart
    /etc/rc.d/rc.openvpn@<em>connection-name</em> reload
    /etc/rc.d/rc.openvpn@<em>connection-name</em> reconnect
    /etc/rc.d/rc.openvpn@<em>connection-name</em> status</code></pre>
Without the <code>@<em>connection-name</em></code> part, the commands apply to all connection files under
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
To enable a service and start it automatically with Slackware, append the following lines to file `/etc/rc.d/rc.local`:
<pre><code>    if [ -x /etc/rc.d/rc.openvpn@<em>connection-name</em> ]
    then
	/etc/rc.d/rc.openvpn@<em>connection-name</em> start
    fi</code></pre>
Append the following lines to file `/etc/rc.d/rc.local_shutdown`:
<pre><code>    if [ -x /etc/rc.d/rc.openvpn@<em>connection-name</em> ]
    then
	/etc/rc.d/rc.openvpn@<em>connection-name</em> stop
    fi</code></pre>

Remember to make the `rc.local_shutdown` file executable:
```sh
    chmod +x /etc/rc.d/rc.local_shutdown
```

This init script will  see if a connection is enabled this way when you run the `status` command.
But currently it is not possible to enable a connection automatically, so this has to be done manually.

For the same purpose, you can also create systemv style links instead, under the
`/etc/rc.d/rc[0-6KSs].d/` directories and the link from `rc3.d/` will be seen by this init
script. But remember the native method in Slackware, to enable a service, is to 
add it to the `rc.local` and `rc.local_shutdown` files.

## Features
All connections are created with the `openvpn` command:
```sh
    /usr/sbin/openvpn --cd "/etc/openvpn/" --config "$CONFIG_FILE" --daemon
```
Other options like <code>syslog <em>tunnel-name</em></code> must be included in the given config file
(under `/etc/openvpnv/`).

Some script features include:
 * Highlighting for command output in the console
 * Support for multiple openvpn connections if needed
     * Operate on either all connections at once, or on each connection at a time
 * Include openvpn statistics for the connection in the status output
