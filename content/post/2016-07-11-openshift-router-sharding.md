---
author: Kent Hua
comments: true
date: "2016-07-11T13:00:00Z"
modified_time: "2016-07-11T17:00:19.397-07:00"
tags: [openshift, router, sharding, 3.2]
title: OpenShift Container Platform 3.2 - router sharding
---

Tested: OCP 3.2.1.4

So I decided to give router sharding a try in the new OpenShift Container Platform 3.2 release.  This is just a quick way to get started along with some documentation links.  As with OpenShift 3.x, the DNS component is left up to you.  The [origin documentation](https://github.com/openshift/origin/blob/master/docs/router_sharding.md) has some details on this.

First thing you'll want to be aware of is, ideally this will be setup on different infra nodes and/or host ips (I haven't tested this with host ips, just a though) because each router shard instance is going to require it's set of hostPorts by default, needing 80, 443, and 1936.  

My example quick setup only has 1 infra node.  This quick setup will use the same wildcard DNS and I just modify ports accordingly.

Check out the documentation provided on [router shards](https://docs.openshift.com/enterprise/3.2/install_config/install/deploy_router.html#creating-router-shards).  Described below are two methods.  Though method 2 is cleaner, depending on your environment.

### Method 1
Use the base script `mkshard` described in the documentation.  

Let's create some shards, (I tweaked the `replicas=1`):  
{{< highlight shell >}}
oc project default
./mkshard.sh 1 sla=high
./mkshard.sh 2 sla=low
{{< / highlight >}}

The number is the shard number and the portion after is the applied `ROUTE_LABELS` that this particular shard that your application project/routes will need to use this particular shard.

You will notice that your pods will likely fail to deploy with a status of `Pending`.  If you describe the pod, you will likely see for your infra node, `PodFitsPorts`.  Since the two new router shards want the port taken by your default router 80, 443 and 1936.  This is where different infra nodes and/or additional IPs will help give us access to those default ports.  

So how do we get the current setup running?  

* Update the dc/router-shard-1: `oc edit dc/router-shard-1`  
  * Modify all the 80->81, 443->444, 1936->1937  
* Update the dc/router-shard-2: `oc edit dc/router-shard-2`   
  * Modify all the 80->82, 443->445, 1936->1938  
* You may need to delete some of the older failed deploy / router pods.   

### Method 2 
An example version of the `mkshard.sh` script above to tweak the ports accordingly:
{{< highlight shell >}}
#!/bin/bash
# Usage: mkshard ID SELECTION-EXPRESSION HTTP_PORT HTTPS_PORT STATS_PORT
id=$1
sel="$2"
http_port=$3
https_port=$4
stats_port=$5
router=router-shard-$id           
oc adm router $router --replicas=0 --ports=$http_port:$http_port,$https_port:$https_port --stats-port=$stats_port
dc=dc/router-shard-$id            
oc env $dc ROUTE_LABELS="$sel"  
oc env $dc ROUTER_SERVICE_HTTP_PORT=$http_port
oc env $dc ROUTER_SERVICE_HTTPS_PORT=$https_port
oc scale $dc --replicas=1
{{< / highlight >}}
note: I'm using `oc adm` above instead of `oadm` because my environment is using atomic host and I'm doing everything from a remote machine via `oc`, so I don't have access to an `oadm`  

ex usage of the tweaked script:
{{< highlight shell >}}
$ ./mkshard1.sh 1 sla=med 81 444 1937
{{< / highlight >}}

### Continue here
Some options that I haven't tested:  

* Additional infra nodes  
* Additional IPs for our ports to bind to (would this be even possible?)  

So we got the two router-shard pods up and running.  You will likely need to modify your iptables to open up the new ports on your infra node.
{{< highlight shell >}}
iptables-save > ~/iptables-output
vi ~/iptables-output
iptables-restore < ~/iptables-output
{{< / highlight >}}


One way to get your application to use the new route will be to add an additional label to your route.  In YAML, under the `metadata.labels` of the route, add one of your `ROUTE_LABELS`.  For example `sla: high`.  Your application will now be accessible through port 81, using your new `router-shard-1`.  

To use a router at a project level, you need to apply a project namespace.

1. Add it to your router: `oc env dc/<router_shard> NAMESPACE_LABELS=site=prod` 
2. Add it to your project:  `oc label namespace <project> site=prod`


Observations:

* If you check the log of your new router pod, it should say: `router.go:153] Router is only using routes in namespaces matching site=prod`
* If for some reason it's not working, verify that your project `NAMESPACE_LABELS` and route `ROUTE_LABELS` match the designated router configurations.  Also for existing routes, ensure that your ingress lists the intended `routerName`.  In the UI it's shown as `exposed on router <router_name>`  It may also take time to refresh.
* If your route has a matching router with only `ROUTE_LABELS`, the switch happens quickly.
* For a project defined with a `NAMESPACE_LABELS` matching a router, new applications will pick up the routing quickly.  Existing projects may require tweaking of the route to refresh the routing list.
* For `NAMESPACE_LABELS`, you may also need to add `cluster-reader` permissions to the router service account.  `oc adm policy add-cluster-role-to-user cluster-reader system:serviceaccount:default:router` -- Needs to be verified, reference [here](https://github.com/openshift/openshift-docs/blob/master/install_config/install/deploy_router.adoc#using-namespace-router-shards)

Updating labels:

* To remove the label NAMESPACE_LABELS, do the following: `oc env dc/<router_name> NAMESPACE_LABELS-`
* To remove a project namespace label, `oc label ns <project> site-`
