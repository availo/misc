# WowzaMediaServer VHost-capable LoadBalancer

## Important notice

This code is currently only *very* briefly tested. Please use this
module with extreme caution for the time being, and don't trust it to
do anything you believe it ought to do.

Feel free to send any questions, suggestions or comments to me on
github@segfault.no

## Prerequisites:

### Original LoadBalancer 2.0
This module depends on the original LoadBalancer 2.0 in order to run:
http://www.wowza.com/forums/content.php?108

Pretty much all the code is based on the source files included in the
original addon.

### json-simple
Grab "json-simple" from http://code.google.com/p/json-simple/

The version used while developing this was json-simple-1.1.1.jar, but
newer versions should work as well.

### Optional: Wowza IDE 2 (for compiling / extending)
http://www.wowza.com/media-server/developers#wowza-ide

## Installation instructions

Step 1:
Copy lib/availo-vhostloadbalancer.jar to the [install-dir]/lib/ folder of Wowza
Media Server 2 or 3.

Step 2:
Follow the instructions under "To setup a load balancer 'listener'" in
README.html from the original LoadBalancer 2.0 application.
Download this from http://www.wowza.com/forums/content.php?108 if you haven't
already done so.

Step 3:
Change the <ServerListener><BaseClass> in Server.xml (from "listener"-step 4
in the original README.html) to the following value:
`<BaseClass>com.availo.wms.plugin.vhostloadbalancer.ServerListenerLoadBalancerListener</BaseClass>`

The "loadBalancerListenerRedirectorClass"-property in Server.xml (also from
step 4 in the original documentation) on the LoadBalancer Listener needs to be
updated. New value:
`<Value>com.availo.wms.plugin.vhostloadbalancer.LoadBalancerRedirectorBandwidth</Value>`

Step 4:
In every active VHost.xml file (as defined in VHosts.xml), change the
BaseClass (from step 5 in the original README.html) to the following value:
`<BaseClass>com.availo.wms.plugin.vhostloadbalancer.HTTPLoadBalancerRedirector</BaseClass>`

Change all instances of "enableServerInfoXML" to "enableServerInfo", as this
version has support for more output formats. (JSON)

If you only added the HTTPProvider to one VHost.xml file, you need to duplicate
the config in all your other active vhosts, since this is how the loadbalancer
can determine what VHost to redirect the client to.


## Configuring the LoadBalancerSender

Step 1:
Follow the instructions under "To setup a load balancer 'sender' on an 'edge'
server" from README.html in the original LoadBalancer 2.0 application.
Download this from http://www.wowza.com/forums/content.php?108 if you haven't
already done so.

Step 2:
Change the <ServerListener><BaseClass> (from step 3 in the original
README.html) to the following value:
`<BaseClass>com.availo.wms.plugin.vhostloadbalancer.ServerListenerLoadBalancerSender</BaseClass>`

The "loadBalancerSenderMonitorClass"-property, also in Server.xml, needs to be
updated as well. New value:
`<Value>com.availo.wms.plugin.vhostloadbalancer.LoadBalancerMonitorVhost</Value>`

Optional step 3:
While still in Server.xml, you may also add a different weight for the current
server. The server weight defaults to 1. Weight works by increasing the capacity
a particular server has, compared to other servers.

If all servers has the same capacity, skipping this option completely is fine.

To use a different weight, add the following property to the bottom of Server.xml:
	<Property>
		<Name>loadBalancerSenderServerWeight</Name>
		<Value>5</Value>
		<Type>Integer</Type>
	</Property>

In the example above, the weight of 5 would mean that a particular server can
handle 5 times the traffic compared to a server with the default weight of 1.

If you have three 1Gbps servers and assume that their bottleneck is the network,
and one 10Gbps server that can handle ~6Gbps, you would use a weight of 1 for
the three first servers, and a weight of 6 for the last one.

For two bundled 1Gbps interfaces on one server, you would use a weight of 2.

Please be adviced: a server can *not* have a weight of 0. The correct way
to handle this is to pause (or stop) the server, as described in README.html
from the original LoadBalancer 2.0 module.

Optional step 4:
If you *know* you will only use VHost-aware load balancers, you may remove the
loadBalancerSenderRedirectAddress-property in Server.xml, as this will never
be used as long as all senders and listeners support VHost-loadbalancing.

NB: I have not done extensive testing after removing this value completely.
If you want to be cautious, you may just remove the "[redirect-address]", and
leave it blank.

Step 5:

Add a "loadBalancerVhostRedirectAddress" property to all active VHost.xml files:
	<Property>
		<Name>loadBalancerVhostRedirectAddress</Name>
		<Value>[vhost-ip-address]</Value>
	</Property>

This IP address should be the same as you have defined in the first <HostPort>
section for the respective VHost.

Step 6:
Change the ModuleLoadBalancerRedirector-module to the following:
`<Class>com.availo.wms.plugin.vhostloadbalancer.ModuleLoadBalancerRedirector</Class>`