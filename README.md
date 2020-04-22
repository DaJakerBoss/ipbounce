# ipbounce

ipbounce is an IP-level proxy.
A host asks the bouncer to connect it and another IP, the guest.
After that, any packets that the bouncer receives from the host are forwarded to the guest, and vice versa.
This allows ipbouncers to be agnostic to the actual protocol being used over them.

ipbounce is also version-agnostic; it accepts both IPv4 and IPv6 for control and connections.

Specifically, ipbounce:

- Listens for all incoming IP packets, regardless of protocol
- If the protocol type is 180, follows the instructions
- If the source IP has no associated connection, drops the packet silently
- Changes the "destination" in the packet to the other end of the connection
- Resets the TTL to 255
- Sends the packet to the other end of the connection, otherwise leaves it unchanged

Note that this is currently a project to (re)familiarize myself with sockets and C.
There are **several** gaping security holes.
See **§&nbsp;Security** for more information.

## Setting up a connection

When you want to use the bouncer, send it an IP packet with the protocol type 180 and the following big-endian data:

- The single byte `0x01`
- Guest IP

Note that each IP can only be party to one connection at a time.

## Using a connection

Once a connection has been set up, nothing else is needed to use it.
Use whatever connection you'd have used with the guest, and it will go through transparently.

Note that, because packets are being sent through another host, you may see double or more the packet loss.
This is an unavoidable consequence of routing traffic to an intermediate host and not correctable at the IP level.
However protocols like TCP should still easily accomodate.

## Tearing down a connection

When you're finished with the bouncer, send it an IP packet with the protocol type 180 and the following big-endian data:

- The single byte `0x02`

The connection you set up before will be torn down.

## Security

Because this is (at least currently) a toy project, I've ignored several security holes, both for potential endpoints and the bouncer.

1.  Anyone can request a connection to any IP.
    Any traffic coming into the server that isn't associated with a connection can be sniffed by setting up a  connection to that IP.
2.  Host IPs are not verified.
    Attackers could send spoofed IP packets to set up hundreds of connections without closing them.
3.  Guest IPs are not verified or filtered.
    An attacker could establish a connection through the ipbouncer to the internal network the ipbouncer is on.

None of these would be *impossible* to fix, but it's not worth the effort when I just want to relearn C.
