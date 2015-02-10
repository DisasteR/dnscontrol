# dnscontrol
A script collection to manage a Bind DNS server farm. Using one master and 1 or more slaves.

## Requirement

```
awk grep egrep sed
```

## Installation

```
# cd /opt
# git clone https://github.com/DisasteR/dnscontrol.git

# cd /etc/bind
# ln -s /opt/dnscontrol/named.conf.master-zones .
```

edit *dnscontrol.cfg*

### Suggested Bind Configuration

#### Master

***named.conf :***

```
// This is the primary configuration file for the BIND DNS server named.
//
// Please read /usr/share/doc/bind9/README.Debian.gz for information on the 
// structure of BIND configuration files in Debian, *BEFORE* you customize 
// this configuration file.
//
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/rndc.key";
include "/etc/bind/named.conf.logging";
include "/etc/bind/named.conf.controls";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
include "/etc/bind/named.conf.master-zones";
```

***rndc.key :***

```
key "rndc-key" {
    algorithm hmac-md5;
    secret "__YOUR_KEY__";
};
```

***named.conf.options :***

```
options {
    directory "/var/cache/bind";

    // If there is a firewall between you and nameservers you want
    // to talk to, you may need to fix the firewall to allow multiple
    // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

    // If your ISP provided one or more IP addresses for stable 
    // nameservers, you probably want to use them as forwarders.  
    // Uncomment the following block, and insert the addresses replacing 
    // the all-0's placeholder.

    // forwarders {
    //  0.0.0.0;
    // };

    //========================================================================
    // If BIND logs error messages about the root key being expired,
    // you will need to update your keys.  See https://www.isc.org/bind-keys
    //========================================================================
    dnssec-validation auto;

    auth-nxdomain no;    # conform to RFC1035
    listen-on-v6 { any; };

    notify yes;
    also-notify { 192.95.25.138; };

    allow-transfer { 127.0.0.1; __MASTER_IP__; __SLAVES_IP__; };

    allow-recursion { 127.0.0.1; __ALLOWED_RECUSION_IP__; };

};

// ======================
// Repeat for each Slave
// ======================
server __SLAVE_IP__ {
        keys { "rndc-key"; };
};
```

***named.conf.controls :***

```
controls {
    inet * allow { 127.0.0.1; __MASTER_IP__; } keys { "rndc-key"; };
};
```

***named.conf.logging :***
```
logging {

    channel bind-log {
    file "/var/log/named.log" versions 3 size 5m;
        print-category yes;
        print-severity yes;
        print-time yes;
        severity info;
    };
    category default { bind-log; };
    category update { bind-log; };
    category update-security { bind-log; };
    category security { bind-log; };
    category queries { bind-log; };
    category lame-servers { null; };

        channel xfer-log {
                file "/var/log/named-xfer.log";
                print-category yes;
                print-severity yes;
                print-time yes;
                severity info;
        };
        category xfer-in { xfer-log; };
        category xfer-out { xfer-log; };
        category notify { xfer-log; };
};
```

#### Slaves

***named.conf :***

```
// This is the primary configuration file for the BIND DNS server named.
//
// Please read /usr/share/doc/bind9/README.Debian.gz for information on the 
// structure of BIND configuration files in Debian, *BEFORE* you customize 
// this configuration file.
//
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/rndc.key";
include "/etc/bind/named.conf.logging";
include "/etc/bind/named.conf.controls";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
include "/etc/bind/named.conf.slaves-zones";
```

***rndc.key (Same as master):***

```
key "rndc-key" {
    algorithm hmac-md5;
    secret "__YOUR_KEY__";
};
```

***named.conf.options :***

```
options {
    directory "/var/cache/bind";

    // If there is a firewall between you and nameservers you want
    // to talk to, you may need to fix the firewall to allow multiple
    // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

    // If your ISP provided one or more IP addresses for stable 
    // nameservers, you probably want to use them as forwarders.  
    // Uncomment the following block, and insert the addresses replacing 
    // the all-0's placeholder.

    // forwarders {
    //  0.0.0.0;
    // };

    //========================================================================
    // If BIND logs error messages about the root key being expired,
    // you will need to update your keys.  See https://www.isc.org/bind-keys
    //========================================================================
    dnssec-validation auto;

    auth-nxdomain no;    # conform to RFC1035
    listen-on-v6 { any; };

    allow-transfer { 127.0.0.1; __MASTER_IP__; __SLAVES_IP; };

    allow-recursion { 127.0.0.1; __ALLOWED_RECUSION_IP__; };
};

server __MASTER_IP__ {
        keys { "rndc-key"; };
};
```

***named.conf.controls :***

```
controls {
    inet * allow { 127.0.0.1; __MASTER_IP__; } keys { "rndc-key"; };
};
```

***named.conf.logging (Same as master):***
```
logging {

    channel bind-log {
    file "/var/log/named.log" versions 3 size 5m;
        print-category yes;
        print-severity yes;
        print-time yes;
        severity info;
    };
    category default { bind-log; };
    category update { bind-log; };
    category update-security { bind-log; };
    category security { bind-log; };
    category queries { bind-log; };
    category lame-servers { null; };

        channel xfer-log {
                file "/var/log/named-xfer.log";
                print-category yes;
                print-severity yes;
                print-time yes;
                severity info;
        };
        category xfer-in { xfer-log; };
        category xfer-out { xfer-log; };
        category notify { xfer-log; };
};
```
