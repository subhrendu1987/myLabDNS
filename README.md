# Environment config
## Install docker

## Tutorial
[Video](https://www.youtube.com/watch?v=syzwLwE3Xq4)

## Docker references
[Cheat sheet](https://github.com/subhrendu1987/DockerCommandReferences/blob/main/README.md)

## Download ubuntu/bind9 from dockerhub
`sudo docker pull ubuntu/bind9`

## Stop local resolver
Edit `/etc/systemd/resolved.conf` and add `DNSStubListener=no`


# Configuration
##
Used topology in this example
```
<mobile 192.168.0.104> --- <Router 192.168.0.1> --- <DNS serv 192.168.0.103>
```

## docker-compose.yaml
Modify ports directive and use your own IP (i.e. in my case 192.163.0.103)
```
        ports:
            - "<IP address of system>:53:53/tcp"	
            - "192.168.0.103:53:53/udp"
``` 
## config/named.conf
### Access control list:
Allow internal network entries
```
acl internal{
	192.168.0.0/16; # All internal IPs 
};
```

### Zone records:
Change the zone names and file name as per your requirements
```
zone "uclnetlocal.idrbt.ac.in" IN{
	type master;
	file "/etc/bind/mapping.zone";
};
```

## config/mapping.zone:
### ORIGIN:
```
$ORIGIN uclnetlocal.idrbt.ac.in.
```
### SOA and nameserver:
Use serial number as date of modification
```
@	IN	SOA	ns.uclnetlocal.idrbt.ac.in.	5gucl.uclnetlocal.idrbt.ac.in (	; SOA record
							06012023	;Serial No. /Date
							12h		;refresh time
							15m		;retyr time
							3w		;expire
							2d		;TTL
							)
	IN	NS	ns.uclnetlocal.idrbt.ac.in.						; Name server record
ns	IN	A	192.168.0.103								; Resolve IP for name server
```
### DNS entries
```
mobile		IN	A	192.168.0.154
*.mobile	IN	A	192.168.0.154		; e.g. resolve abc.mobile 
```
### Test DNS
```
nslookup www.google.com 192.168.0.103 					# Success implies forwarding is fine
nslookup mobile.uclnetlocal.idrbt.ac.in 192.168.0.103 	# Success implies local resolution is fine
nslookup abc.mobile.uclnetlocal.idrbt.ac.in 192.168.0.103
```