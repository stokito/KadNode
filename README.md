## kadnode(1) - P2P name resolution daemon

## SYNOPSIS

`kadnode`  [--value-id identifier] [--port port] [--daemon] [...]

`kadnode-ctl`  [...]

## DESCRIPTION

**KadNode** is a small P2P DNS resolution daemon based on a Distributed Hash Table (DHT).
It can intercept and resolve DNS request of a given top level domain in the background of the system.
This makes it usable even in a browser or on the command line. KadNode can serve as
as a personal and decentralized DynDNS service that way. A KadNode instance can also
announce domains to the network. A cryptographic public key can be used to let KadNode
filter out nodes that announce the same domain as the expected one.
The DHT is identical to the one used in the Transmission Bittorrent client and works
on the Internet as well as on local networks.
Packages for multiple operating systems are available to setup your system right away.
Have a look at the configuration file (/etc/kadnode/kadnode.conf) for further instructions.

Features:

* IPv4/IPv6 support
* UPnP/NAT-PMP support
* local peer discovery
* small size 75KB-125KB
* public/secret key authentication ([libsodium](https://github.com/jedisct1/libsodium))
* command line interface (kadnode-ctl)
* NSS support through /etc/nsswitch.conf
* integrated simplified DNS server (supports A, AAAA, and SRV requests)
* packages for MacOSX/Debian/FreeBSD/OpenWrt/ArchLinux
* peer file import/export on startup/shutdown

Most features are optional and can be left out to reduce the binary size.


## JOIN THE SWARM

If KadNode cannot join a swarm (an empty peer file and no other peer on the local network),
then a node address is needed to boostrap from. To do this, insert *bttracker.debian.org*
into an empty file and provide it to KadNode:

```
kadnode --peerfile peers.txt
```

At most 150 good peers are written back to the peer file on KadNode shutdown and
also every 24 hours. This ensures successful boostrapping on the next startup.
The peer file is not written after at least 5 minutes of runtime.

## AUTHENTICATION

For most use cases you first need to create a secret/public key pair:

```
$kadnode --auth-genkey
public key: <public-key>
secret key: <secret-key>
```

(The keys are not displayed for convenience.)

### EXAMPLE 1

A typical use case would be to reach a computer (Node1) from another (Node2).
On Node 1 we announce an identifier (`sha1(myname)`):
```
$kadnode --value-id myname.p2p
```

On other computers running KadNode, we can use myname.p2p to resolve
the IP address of Node1 (e.g. in the Browser). It may take ~8 seconds on the first try.
The problem with this approach is that others could also announce the same identifier.
This would result in multiple IP addresses to be found and used.
To avoid this you can use secret/public key pairs. See the next examples.

### EXAMPLE 2

Now we want to reach a computer (Node1) from another (Node2) using cryptographic keys.
We tell KadNode on Node1 to *announce* its secret key.
```
$kadnode --value-id <secret-key>.p2p
```
What is actually announced is the SHA1 hash of the derived public key (`sha1(<public-key>)`),
not the secret key itself.

On Node2, we can now resolve `<public-key>.p2p`.

### EXAMPLE 3

Instead of using keys directly as in example 1, we can also use domains.

On Node1, well tell Kadnode to announce that we have node1.p2p using the secret key.
```
$kadnode --auth-add-skey "node1.p2p:<secret-key>" --value-id node1.p2p
```
The announced identifier here is `sha1(<public-key>+"node1")`.

On Node2, we tell the node to use the public key to verifiy requests for node1.p2p.
```
$kadnode --auth-add-pkey "node1.p2p:<public-key>"
```
The identifier that is searched for is `sha1(<public-key>+"node1")`.

On Node2, we now can enter node1.p2p into the browser or try to ping node1.p2p to see
if the address is resolved successfully. The authentication step uses the public key
to filter out all nodes that do not know the secret key.

### EXAMPLE 4

Instead of just one name, it is possible to use patterns with a wildcard in front:

Node1 will be reachable using node1.p2p and foobar.p2p.
```
$kadnode --auth-add-skey "*.p2p:<secret-key>" --value-id node1.p2p --value-id foobar.p2p
```

Node2 will be reachable using node2.p2p and foobar.p2p as well.
```
$kadnode --auth-add-skey "*.p2p:<secret-key>" --value-id node2.p2p --value-id foobar.p2p
```

On Node3, resolving "node1.p2p", "node2.p2p" and "foobar.p2p" to its IP address should now work.
```
$kadnode --auth-add-pkey "*.p2p:<public-key>"
```
Since foobar.p2p is used twice, KadNode will give both IP addresses.
But almost all programs will just use the first address they get,
which is the first that will be successfully verified.

Multiple --auth-add-key and multiple --value-id arguments are possible.
Pattern conflicts will result in an error message on startup.
Command line options can also be put inside a configuration file (see --config *file*).

## IDENTIFIERS

Identifiers are what you use to lookup IP addresses (e.g. "foo.p2p").
They will reduced to 20 Byte representations based on the SHA-1 message digest algorithm.
The digest is what will be then used in the actual lookup process.
KadNode allows four types of identifiers:

* Raw identifiers are 20 Byte identifiers written as hex strings. No digest will be applied.
  *  E.g: `<40_hex_characters>.p2p`

* Raw public key identifiers are 32 Byte hex strings. They will be interpreted as a public key.
  * The SHA-1 digest of the key string respresentation is used to locate nodes.
  * The public key is used to verify that the found nodes have the corresponding secret key.
  * E.g.: `<64_hex_characters>.p2p`

* Plain identifiers are just strings that have no key associated to them.
  * The SHA-1 digest of the string is used to find nodes.
  * E.g.:  `example.p2p`

* Plain identifiers that match a given pattern and have a public key assigned to them.
  * The SHA-1 digest of the string and public key is ued to find nodes.
  * The public key is used to verify if the nodes found have the corresponding secret key.
  * E.g.:  `foo.p2p`

All identifiers are converted to lowercase and are therefore case insensitive.
A ".p2p" at the end of every identifier is ignored. It is used to direct requests to KadNode.


## OPTIONS
  * `--node-id` *id*  
    Set the node identifier. This option is rarely needed.  
	By default the node id is random.

  * `--value-id` *id[:port]*  
    Add a value identifier and optional port to be announced every 30 minutes.  
    The announcement will associate this nodes IP address with this identifier.  
    This option may occur multiple times.

  * `--peerfile` *file-path*  
    Import peers for bootstrapping and write good peers  
	to this file every 24 hours and on shutdown.

  * `--user` *name*  
    Change the UUID after start.

  * `--port` *port*  
    Bind the DHT to this port.  
    Default: 6881

  * `--config` *file*  
    Provide a configuration file with one command line  
    option on each line. Comments start after '#'.

  * `--ifce` *interface*  
    Bind to this specific interface.

  * `--fwd-disable`  
    Disable UPnP/NAT-PMP to forward router ports.

  * `--daemon`  
    Run in background.

  * `--query-tld` *domain*  
    Top level domain used to filter queries to be resolved by KadNode. (Default: ".p2p")

  * `--verbosity` *level*  
    Verbosity level: quiet, verbose or debug (Default: verbose).

  * `--pidfile` *file-path*  
    Write process pid to a file.

  * `--lpd-addr` *address*  
    Send LPD packets to this multicast address as long no peers were found.  
    Default: 239.192.202.7:6771 / [ff08:ca:07::]:6771

  * `--lpd-disable`  
    Disable Local Peer Discovery (LPD).

  * `--cmd-disable-stdin`  
    Disable the local control interface.

  * `--cmd-port` *port*  
    Bind the remote control interface to this local port (Default: 1700).

  * `--dns-port` *port*  
    Bind the DNS server to this local port (Default: 5353).

  * `--nss-port` *port*  
    Bind the "Name Service Switch" to this local port (Default: 4053).

  * `--web-port` *port*  
    Bind the web server to this local port (Default: 8053).

  * `--auth-gen-keys`  
    Generate a secret/public key pair.

  * `--auth-add-pkey` [*pattern*:]*public-key*  
    Associate a public key with any value id that matches the pattern.  
    Used to verify that the other side has the secret key.  
    This option can occur multiple times.

  * `--auth-add-skey` [*pattern*:]*secret-key*  
    Associate a secret key with any value id that matches the pattern.  
    Used to prove the ownership of the domain.  
    This option can occur multiple times.

  * `--mode` *protocol*  
    Enable IPv4 or IPv6 mode for the DHT (Default: ipv4).

  * `-h`, `--help`  
    Print the list of accepted options.

  * `-v`, `--version`  
    Print program version and included features.

## kadnode-ctl

**kadnode-ctl** allows to control KadNode from the command line.

  * `-p` *port*  
    The port used to connect to the command line of a local KadNode instance (Default: 1700).

  * `-h`  
    Print this help.

### KadNode Console Commands

  * `status`  
    Print the node id, the number of known nodes / searches / stored hashes and more.

  * `lookup` *query*  
    Lookup the IP addresses of all nodes that claim to satisfy the query.  
	The first call will start the search.

  * `announce` [*query*[<i>:*port*</i>] [<i>*minutes*</i>]]  
    Announce that this instance is associated with a query  
    and an optional port. The default port is random (but not equal 0).  
    No *minutes* trigger a single announcement. Negative *minutes*  
    last for the entire runtime. Otherwise the lifetime is set *minutes* into the future.  
    No arguments will announce all identifiers at once.

  * `import` *addr*  
    Send a ping to another KadNode instance to establish a connection.

  * `export`  
    Print a few good nodes.

  * `list` [`blacklist`|`buckets`|`constants`|`forwardings`|`results`|`searches`|`storage`|`values`]  
    List various internal data structures.

  * `blacklist` *addr*  
    Blacklist a specifc IP address.

## Web Interface

The optional web interface allows queries of these forms:

  * `http://localhost:8053/lookup?foo.p2p`
  * `http://localhost:8053/announce?foobar`
  * `http://localhost:8053/blacklist?1.2.3.4`

If the interface cannot be reached then the interface might be disabled (port set to 0)
or not compiled in (check `kadnode --version`).
In case the IPv6 entry for localhost is not used or missing, try `[::1]` instead of `localhost`.

## PORT FORWARDINGS

If KadNode runs on a computer in a private network, it will try to establish a port forwarding for the DHT port.
Port forwarding only works if UPnP/NAT-PMP is compiled into KadNode and is supported by the gateway/router.
Also, ports attached to announcement values (e.g. `--value-id foo.p2p:80`) will result in additional port forwardings.
This is useful to make a local service (e.g. web server) reachable from the Internet.

## NOTES

  * *.p2p* at the end of an identifier (or set by --query-tld) is ignored by KadNode. It is used to filter requests and divert them to KadNode.
  * The interfaces (NSS, DNS, command line) may return the localhost address if the node itself announced a value.

## LICENSE

  MIT/X11

## AUTHORS

  * KadNode: Moritz Warning (http://github.com/mwarning)
  * Kademlia: Juliusz Chroboczek
  * SHA-1: Steve Reid
