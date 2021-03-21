## Routing tips for VPNs on OS X

When VPNs just work, they're a fantastic way of allowing access to a private network from remote locations. When they don't work it can be an experience in frustration. I've had situations where I can connect to a VPN from my Mac, but various networking situations cause routing conflicts. Here are a couple of cases and how I've been able to get around them.

### Specific cases

#### Case 1: conflicting additional routes.

In this example the VPN we are connecting to has a subnet that does not conflict with our local IP, but has additional routes that conflict in some way with our local network's routing. In my example the remote subnet is 10.0.x.0/24, my local subnet is 10.0.y.0/24, and the conflicting route is 10.0.0.0/8. Without the later route, I can't access all hosts on the VPN without manually adding the route after connecting to the VPN:

    sudo route add -net 10 -interface ppp0
    
In the above case the VPN is a PPTP VPN that uses ppp0 as the network interface. With this additional route, I can now access all the hosts I need to on the VPN. This won't solve the case of trying to access addresses on the 10.0.y.0/24 subnet though.

#### Case 2: conflicting subnet between VPN and local network.

Fairly often a VPN on a private address space subnet can end up conflicting with a local subnet. For example if both the remote and local networks share the 192.168.0.0/24 subnet then our VPN connection ends up being pretty useless as all of the remote addresses will end up being routed to the local network device.

It is possible to get around this in some cases as long as the VPN IP address doesn't conflict directly with a local IP address that you need access to. In this case we need to add a specific route for the remote IP:

    sudo route add -host 192.168.0.x -interface tun0
    
In the above case I'm routing the host 192.168.0.x (replace the x with your specific address) via the tun0 device (in this case an OpenVPN connection).

### Useful commands to debug routing issues.

The following command will show the existing routing table (IPv4 only):

    netstat -nr -f inet
    
The following command will show you how a specific host will get routed:

    route get HOSTNAME_OR_IP
