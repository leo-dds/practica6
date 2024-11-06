[# practica6
](https://github.com/leo-dds/practica6.git)

1.-Creo la red con docker network create bind9_subnet  

2.-Creo un contenedor con ducker run -it ubuntu, conecto el contenedor a la red con docker network connect bind9_subnet ubuntu.

3.-Con docker network ispect bind9_subnet nos proporciona informacion sobre la red:
~~~
 docker network inspect bind9_subnet
[
    {
        "Name": "bind9_subnet",
        "Id": "4d9b24ea47ec859e1a59819c1b32cc19c5ee343794b8dc8bffc906099774d8ee",
        "Created": "2024-11-05T22:53:17.132470725+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
~~~
4.- Fichero bind9_subnet.json
~~~
sri@sri-VirtualBox:~/docker/practica-3/bind9-container/bind9-project$ cat bind9_subnet.json 
[
    {
        "Name": "bind9_subnet",
        "Id": "4d9b24ea47ec859e1a59819c1b32cc19c5ee343794b8dc8bffc906099774d8ee",
        "Created": "2024-11-05T22:53:17.132470725+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
~~~

5.- Ficheros de configuración de Docker

  5.1.-Configuración de zona local
 ~~~
  sri@sri-VirtualBox:~/Docker/config$ cat ~/Docker/config/named.conf.local
zone "asircastelao.int" {
	type master;
	file "/var/lib/bind/db.asircastelao.int";
	allow-query {
		any;
	};
};
~~~
  5.2.-Zonas por defecto
~~~
cat ~/Docker/config/named.conf.default-zones
// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};

  5.3.-Archivo named.conf
  cat ~/Docker/config/named.conf
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
~~~
5.4.- Archivo db.127
~~~
cat ~/Docker/config/db.127
    604800
@       IN      SOA     localhost. root.localhost. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
1       IN      PTR     localhost.

~~~
6.-Iniciar docker-compose
~~~
sri@sri-VirtualBox:~/docker/practica-3/bind9-container/bind9-project$ docker-compose up -d
[+] Running 6/6
 ✔ cliente 1 layers [⣿]      0B/0B      Pulled                         8.0s 
   ✔ ff65ddf9395b Pull complete                                        2.6s 
 ✔ asir_bind9 3 layers [⣿⣿⣿]      0B/0B      Pulled                    9.5s 
   ✔ 63d049b2edcb Pull complete                                        2.8s 
   ✔ 628b314bec1f Pull complete                                        1.7s 
   ✔ d8dd8249bef1 Pull complete                                        0.5s 
network bind9_subnet declared as external, but could not be found
~~~

7.-Inicio de Contenedor Cliente e Instalación de Git y Bind
~~~
docker run -it --rm --privileged ubuntu:latest bash

root@b7f5961db912:/# echo "nameserver 8.8.8.8" > /etc/resolv.conf
root@5bf5c2e5f5b8:/# apt update
root@5bf5c2e5f5b8:/# apt install dnsutils -y 

root@5bf5c2e5f5b8:/# dig google

; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> google
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1405
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1220
; COOKIE: 40540368dc4427e0eb832e89672bc672d14e22d61f683d54 (good)
;; QUESTION SECTION:
;google.				IN	A

;; AUTHORITY SECTION:
google.			5	IN	SOA	ns-tld1.charlestonroadregistry.com. cloud-dns-hostmaster.google.com. 1 21600 3600 259200 900

;; Query time: 10 msec
;; SERVER: 8.8.8.8#53(8.8.8.8) (UDP)
;; WHEN: Wed Nov 06 19:41:38 UTC 2024
;; MSG SIZE  rcvd: 161



