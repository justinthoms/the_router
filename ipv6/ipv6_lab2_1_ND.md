# IPV6 Lab #2.1. Testing the Neigbor discoverty protocol and the neigbor cache states.

lab#1 test network consists of the_router and a host supporting ipv6

## 0. The neighbor cache is empty

	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static

## 1. ping nonexistent ipv6 address. INCOMPLETE neighbor cache entries.

A neihgbor cache entry with INCOMPLETE state is created.
Incomplete entry lives as long as the probing process has not finished.
During the probing process a number of Neigbor Solicitation 
messages is sent then if a Neighbor Advertisement messages is received 
the entry's state is changed to RECHABLE or the entry is deleted.

	h5 ~ # rcli ping6 -c1 fe80::d6ca:6dff:fe7c:d0d1
	Ping fe80::d6ca:6dff:fe7c:d0d1 56(84) bytes of data.
	timeout
	---
	Ping fe80::d6ca:6dff:fe7c:d0d1 statistics:
	sent: 1, recv: 0 (0%), lost: 1 (100%)
	round-trip min/avg/max = 0.0/0.0/0.0
	h5 ~ #
	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static
	port 0, vid 0.3, type 0 fe80::d6ca:6dff:fe7c:d0d1       00:00:00:00:00:00       incomplete      P

Each subsequent probe is sent after waiting the RETRANS_TIME time, so an incomplete 
entry might live no longer than NUMBER_OF_PROBES * RETRANS_TIME; 

Entry fe80::d6ca:6dff:fe7c:d0d1 is deleted:

	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static

## 2. Ping an existing IPv6 address. RECHABLE neighbor cache entries.

The probing process for ip6 address fe80::d6ca:6dff:fe7c:d0dc 
has been completed succesfuly and the RECHABLE entry has been created.

	h5 ~ # rcli ping6 -c1 fe80::d6ca:6dff:fe7c:d0dc
	Ping fe80::d6ca:6dff:fe7c:d0dc 56(84) bytes of data.
	reply 56 bytes icmp_seq=1 time=0.337 ms
	---
	Ping fe80::d6ca:6dff:fe7c:d0dc statistics:
	sent: 1, recv: 1 (100%), lost: 0 (0%)
	round-trip min/avg/max = 0.337/0.337/0.337
	h5 ~ #
	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static
	port 0, vid 0.3, type 0 fe80::d6ca:6dff:fe7c:d0dc       D4:CA:6D:7C:D0:DC       reachable       P

## 3. STALE entries.

A rechable entry becomes stale when there are no packets interested in it.
This happens after about half of the entry TTL time has been expired.
Then, after another half TTL entry has been expired the entry is deleted
provided that there is still no traffic. Stale entry is a neighbor cache
entry that stores the link-layer address that might be outdated, but it is
still can be used.

Let's wait some time and output the neibour cache after ping command 
from step 2.
	
	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static
	port 0, vid 0.3, type 0 fe80::d6ca:6dff:fe7c:d0dc       D4:CA:6D:7C:D0:DC       stale   P

Let's wait again. Entry is deleted.

	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static


## 4. Updating STALE entries

Create a STALE entry by executing the ping command and waiting the half TTL.
Then ping the ipv6 address again it entry will become rechable again.
	
	h5 ~ # rcli ping6 -w -c4 fe80::d6ca:6dff:fe7c:d0dc
	Ping fe80::d6ca:6dff:fe7c:d0dc 56(84) bytes of data.
	reply 56 bytes icmp_seq=1 time=0.897 ms
	reply 56 bytes icmp_seq=2 time=0.321 ms
	reply 56 bytes icmp_seq=3 time=0.311 ms
	reply 56 bytes icmp_seq=4 time=0.158 ms
	---
	Ping fe80::d6ca:6dff:fe7c:d0dc statistics:
	sent: 4, recv: 4 (100%), lost: 0 (0%)
	round-trip min/avg/max = 0.158/0.421/0.897
	h5 ~ #
	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static
	port 0, vid 0.3, type 0 fe80::d6ca:6dff:fe7c:d0dc       D4:CA:6D:7C:D0:DC       reachable       P
	h5 ~ #
	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static
	port 0, vid 0.3, type 0 fe80::d6ca:6dff:fe7c:d0dc       D4:CA:6D:7C:D0:DC       stale   P
	h5 ~ #
	h5 ~ # rcli ping6 -w -c4 fe80::d6ca:6dff:fe7c:d0dc
	Ping fe80::d6ca:6dff:fe7c:d0dc 56(84) bytes of data.
	reply 56 bytes icmp_seq=1 time=0.633 ms
	reply 56 bytes icmp_seq=2 time=0.358 ms
	reply 56 bytes icmp_seq=3 time=0.324 ms
	reply 56 bytes icmp_seq=4 time=0.176 ms
	---
	Ping fe80::d6ca:6dff:fe7c:d0dc statistics:
	sent: 4, recv: 4 (100%), lost: 0 (0%)
	round-trip min/avg/max = 0.176/0.372/0.633
	h5 ~ #
	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static
	port 0, vid 0.3, type 0 fe80::d6ca:6dff:fe7c:d0dc       D4:CA:6D:7C:D0:DC       reachable       P

## 5. Stale entries 2.

Replying to echo request packets is not implemented yet.
So, when a ipv6 enabled host pings a router's ipv6 address it
will not get echo reply back, but it will get router MAC address in it's cache.
And the_router itself creates a neigbor cache entry for the host originated 
ping packets. Created neighbor cache entry according to the RFC 4861 
will have STALE state

on a windos machine that have fe80::24e6:3555:bcec:4a36 ipv6 address

	C:\Users\alexk>ping fe80::6a05:caff:fe30:6180
	

	h5 ~ # rcli sh ipv6 arp
	vif     ip      mac     state   flags
	flags: R - isRouter, P - probing, S - static
	port 0, vid 0.3, type 0 fe80::24e6:3555:bcec:4a36       F8:32:E4:72:61:1B       stale   P
