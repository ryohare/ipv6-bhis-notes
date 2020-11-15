# ipv6-bhis-notes
## Overview
These are notes about IPv6 from BHIS's presentation on [IPv6 by Joff Thyer](https://www.blackhillsinfosec.com/webcast-ipv6-how-to-securely-start-deploying/).
# Notes
## Eliminates bolt on protocols
* ARP
## Remediates global route table issues
* Removes CIDR
* All address are internet addresses like original IPv4
* NAT
## Security
* **No more or less secure than IPv4**
* Different however

## Addressing
### 128 Bit addresses
* 2^128
* Not IPv4 with bigger addresses
### Expressed as Hex
* groups of 4 hex digits
### Address Summarization
* 2001:0410:0009:0479:0000:0000:0000:0001
* Same as 2001:410:9:479::1

### Address Format
* First 64 bits- Network Prefix (network address)
* Last 64 bits - Interface ID (host address)
* /48 = 48 bits of the network prefix - largest allocations
	* Subnet furthermore to /64s

### Packets
* 0x86DD -IPv6 upper level (L2 ethernet)
* IPv6 tunneled inside IPv4 for non-native v6 devices
	* 6 in 4
* Ethernet to IPv6 addresses via ICMPv6
	* ICMPv6 - multicast for neighbor and router discovery
	* Multicast = broken - no v6

### Address Types
* Anycast
	* Same address to more than one interface routed to nearest one
		* Multihomed systems with different ISP for example
* Unicast
	* One sender, one recipient address
* Multicast
	* Same as IPv4
* No broadcast anymore - only multicast

## Address Scopes
* Link Local Scope
	* Only directly attached network links
	* Prefix FE80::10
* Global
	* Globally routable
		* Unicast for anycast
* Multicast
	* 4 least sig bits in second octet of IPv6 Adress
	* Format
		* FF0_SCOPE_::
			* _SCOPE_ is the scope of the multicast address
			* Various predefined ones
* Mutlicast Scopes
	* FF00:: Unused
	* FF02:: local/host bound
	* FF02:: link local
		* Area to secure
	* FF03:: realm local
		* Area to secure
		* Defines application area, not organizational area
	* FF04:: admin local
		* Area to secure
	* FF05:: site local
		* Area to secure
	* FF08:: org level
		* Area to secure
	* FF0E:: global
	* FF0F:: Unused
		
## Address Assignment
* SLAAC - auto setup on /64 space
* DHCPv6
* SLAAC and DHCPv6
* Static
* 64 bit subnets commonly deployed
	* Make very spare and widely distributed
	* All ifcs will have to generate link-local scope adrress with fe80::0/64

## Multiple Addresses
* Each ifc on the network can have multiple addresses each in a different scope
	* Link-local
	* Site local
	* Unicast global
* EUI-64 - Extended Use Interface
	* SLAAC global and link-scope local
	* *Chooses address based on ethernet address
		* Splits 48 MAC into 2 24 bit halfs
		* Insert 0cxFFFE in between two components
		* Flip 7th most sig bit from left for universal scope
	* Implication
		* If known ethernet address - know the link scope local address of a machine in a subnet
		* Can also predict global scope address in subnet
			* Only done if SLAAC is used for global
## ICMPv6
* Needed (with multicast) for IPv6 to work, otherwise it will not work
* Same header as v4
* 4 messages types
	* Error
	* Information
	* Neighbor discovery
	* Other
* To protect v6 infrastructure, need to secure ICMP and Multicast
	* Functionally the IPv6 control plane

### ICMP Neighbor Discovery
* Router
	* Like DHCP
		* route/DNS info only
	* Need to prevent hosts from doing this - otherwise steal routes
* Neighbor
	* Like ARP
		* Over multicast
* Redirect		
* Filter all types without an officially required function
	* Otherwise could be covert channels

## Networks
Look at route table to see which / network size you are in for IPv6 - Look for the prefix on the route advertisement
### Why handout so many address
* Because we can
* Doing away with NAT - spread addresses very sparsely to prevent predication
* Can't scan this space - too large
* *No limit to the number of addressed per interface
	* Limit likely encoded into the kernel driver.
### Network Segmentation
* Network segmentation is the same as done with IPv4
	* Companies given /48
	* 80 bits of address space for subdivision
	* Grant subnets in /64
		* Needed for SLAAC to work
		* Otherwise, need to configure DHCP for address negotiation

## Security Concerns
* Bogons
	* IP address which have not been allocated
	* Filter all of these at network perimeter
* IPv6 Address Filtering
	* Anti-Spoofing, done with filtering
	* No packet with a SRC of internal network should be allowed through the perimeter
		* Outside -> Inside
	* No packet with a DEST address of internal network should be allowed through the perimeter
		* Inside -> Outside
* ICMPv6 Perimeter Security
	* Two types
		* Traffic initiated from the perimeter security device
			* Same as transit policy
			* Additional allows
				* Router 133/134
				* Neighbor 135/136
				* Inverse Neighbor
		* Traffic that is in-transit across the perimeter
			* Start with DENY ALL
			* Allow Type 1: Dest Unreachable
			* Allow Type 2: Packet to large
				*MTU discover is part of protocol
			* Allow Type 3 Code 0 Only:
				* Traceroute
			* Allow Type 4 Code
			* Disable ping is desired
			* Type 144-147 only if Mobility Enabled
			* Multicast 151-153
				* Only if global multicast routing
			* Type 137 Redirect
				* Always drop
* Multicast Filtering
	* Differentiate between perimeter from internal network
		* Neighbor discovery has to work internally
		* Router discovery has to work internally
		* End hosts NOT allowed to advertise routes
		* End hosts NOT allowed to gratuitously respond to neighbors
	* Most likely, not participating in global/inter-domain multicast
		* So filter it
		* Any pkt with multicast src address dropped at perimeter
		* Drop unused multicast addresses
		* Block most other multicast addresses leaving the network at the perimeter
* Protocol Normalization
	* Next Header field
		* Extension headers
			* Each one should not appear multiple times in the list
			* List should be in order
			* Additional rules defined in the RFC for ordering the extension headers
		* Chained headers
			* Hop by Hop Proto 0
				* IPv4 Loose Source Routing
	* Like OSI L4 Upper Layer Header
	* Need to normalize the headers into the RFC format
		* Can do this in the perimeter device (if it supports it)
	* RH0 Attack
* Address Privacy and Obsecurity
	* EUI-64
		* 100% reachable for all nodes on the subnet
		* If most of the devices are all made by the same manufacturer, then the upper 24 bits of the MAC are static
			* So the top of the address is the known
		* Know the default middle done for EUI-64
			* SLAAC uses this
		* Leaves only 24 bits of uniqueness on the subnet
			* Only have to search this space for hosts
		* Ping6 -i eth1 ff02::1
			* All addresses in the link-local scope
			* Systems now enumerated
			* Can happen in a dual stack would aswell.
			* Like pinging the broadcast in IPv4 land
	* Privacy Extensions RFC
		* Disable EUI-64
			* Link local address is randomly chosen
		* SLAAC will generate a random global unique address chosen in the /64
			* Duplicate address algorithm run on collisions
	* Endpoint Route Table
		* Populated via ICMPv6 route advertisements
		* Don’t allow endpoints to advertise routers
		* Host based firewall
		* Add to switch network and internal routers
	* NAT66
		* Proposal out - not official yet
		* No reason to do this
			* Enough addresses already
# Summary
* Implement anti-spoof and bogon filtering
* Filter/control ICMPv6 traffic both at perimeter and with LAN
* Don’t allow bogus ICMPv6 Router Advertisements from end nodes
* Enable Privacy extensions (disable EUI-64)
* Assign addresses randomly within the sizable subnets
* SLAAC is really more of an ISP than enterprise org thing
* Don’t use NAT66
* Minimize DoS risk by
	* Choosing perimeter security deices that can normalize protocol extension headers properly
	* NOT trunking VLANs everywhere or network will die.				
* When subnetting less than /64, need to do this along a nibble boundary for routing
	* /64 -> /72 (e.g. 8 more bits)
		* Now have 2^8 subnets if the ISP hands out a /64
		* 64 subnets
	* However, can't use SLAAC for this
	* SLAAC is built under the assumption that:
		* ISPs will allocate a /48 to an org
		* An org will then subnet a /48 into /64'2
			* 2^16 = 65,536 subnets
