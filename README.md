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

If you have a single master, you need to adjust the etc quorum guard. There are two options:

* OCP version 4.3 and before

Edit the clusterversion operator. Create a YAML file with the following contents:

```shell
oc patch clusterversion version --type json -p "$(cat <<- EOF
- op: add
  path: /spec/overrides
  value:
    - group: apps/v1
      kind: Deployment
      name: etcd-quorum-guard
      namespace: openshift-machine-config-operator
      unmanaged: true
EOF
)"
```

Downscale etcd-quorum-guard to one:

```shell
oc scale --replicas=1 deployment/etcd-quorum-guard -n openshift-machine-config-operator
```

* OCP version 4.4 and after




Downscale other operator's pods if necessary when running single master.

```shell
oc scale --replicas=1 deployment.apps/console -n openshift-console

oc scale --replicas=1 deployment.apps/downloads -n openshift-console

oc scale --replicas=1 deployment.apps/oauth-openshift -n openshift-authentication

oc scale --replicas=1 deployment.apps/packageserver -n openshift-operator-lifecycle-manager


# NOTE: When enabled, the Operator will auto-scale this services back to original quantity
oc scale --replicas=1 deployment.apps/prometheus-adapter -n openshift-monitoring
oc scale --replicas=1 deployment.apps/thanos-querier -n openshift-monitoring
oc scale --replicas=1 statefulset.apps/prometheus-k8s -n openshift-monitoring
oc scale --replicas=1 statefulset.apps/alertmanager-main -n openshift-monitoring
```

Check pods not Running or Completed

```shell
oc get pods --no-headers --all-namespaces | egrep -v 'Running|Completed'
```