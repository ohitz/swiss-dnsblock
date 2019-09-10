# Swiss DNS Block

This tool downloads the two official Swiss DNS blacklists and updates
a DNS zone to rewrite requests to these domains to a local webserver.

Blacklists:

* ESBK: https://www.esbk.admin.ch/esbk/de/home/illegalesspiel/zugangssperren/provider.html
* Comlot: https://blacklist.comlot.ch/

The rewriting is done using a BIND "Response Policy Zone" (RPZ). RPZ
is a special DNS zone containing rewrite rules. It can be updated as
any other zone on a master DNS server and transferred to slaves using
the standard zone transfer method.

## Requirements

	apt install curl mpack openssl bind9utils

## Installation

* Copy everything to `/usr/local/swiss-dnsblock`
* Copy `data/config.txt.dist` to `data/config.txt` and edit it
* Copy `data/zone.header.txt` to `data/zone.header` and edit it (note
  that `__SERIAL__` will be replaced by a serial number when the zone
  is updated)
* Setup BIND and configure it as master DNS server (see below)
* Run `swiss-dnsblock update` to download the data files and create
  the zone file. The zone file will be in `data/zone.txt` as well as
  at the location specified in `config.txt`.
* On all DNS resolvers configure the RPZ zone as a slave zone (see
  below).
* When everything works, install a daily cronjob to update the zone if
  necessary: `cp cron/swiss-dnsblock-update /etc/cron.dail/`

## Create RPZ Zone on Master DNS

    acl swiss-dnsblock-slaves {
        IP_OF_SLAVE_DNS/32;
    };

    options {
        ...
        response-policy { zone "rpz-swiss-dnsblock"; };
        ...
    };

    zone "rpz-swiss-dnsblock" {
        type master;
        file "/etc/bind/db.rpz-swiss-dnsblock";
        allow-query { swiss-dnsblock-slaves; };
        allow-transfer { swiss-dnsblock-slaves; };
        also-notify { IP_OF_SLAVE_DNS; };
    };

## Pull RPZ Zone on DNS Slave

    options {
        ...
        response-policy { zone "rpz-swiss-dnsblock"; };
        ...
    };

    zone "rpz-swiss-dnsblock" {
        type slave;
        masters { IP_OF_MASTER_DNS; };
        file "/etc/bind/db.rpz-swiss-dnsblock";
    };
