# OCP4 in libvirt

## Prepare resources

* 1 virtualization host.
    - `libvirt`
    - `ansible`
    - `git`

  In case the host is an additional machine and if it has a graphical environment, I recommend the installation of the remote desktop program `NoMachine` as it has the best peformance. And it can be combined with vnc in order open not only :0 display, but after running `vncserver :1`, `NoMachine` will list two remotes, session 0 and session 1.
    - `tigervnc`
    - `noMachine`
    - `cockpit`

  The idea is to have a helper node with a desktop environment set, then connect to it using the internal libvirt vnc server and work from there.

Within the virtualization host, add a record into the /etc/hosts file for the helper node.

```config
192.168.100.2 helper.lab.io helper
```

Also add the DNS provided by the helper into the resolved config.

```ini
vim /etc/systemd/resolved.conf.d/dns_servers.conf

[Resolve]
DNS=192.168.100.2
Domains=ocp4.lab.io
```
