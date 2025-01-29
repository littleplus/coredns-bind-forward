# coredns-bind-forward
This patch allow DNS query to be forwarded from specific interface's IP

## What's it for
This patch is primarily provided for situations with multiple ISP network connections. Since service providers offer DNS resolution optimization based on ISPs, we need to ensure that DNS queries are made through the designated network interface.

## How to use
1. Clone the coredns's source code and checkout to the commit id `51e11f166ef6c247a78e9e15468647c593b79b9f` or tag `v1.12.0`
```
git clone https://github.com/coredns/coredns.git
cd coredns
git checkout tags/v1.12.0
```

2. Put the code here
```
wget -q https://github.com/littleplus/coredns-bind-forward/raw/refs/heads/main/51e11f166ef6c247a78e9e15468647c593b79b9f.patch
```

3. Apply the patch
```
git apply 
```

4. Build coredns
```
make
```

5. Edit coredns config, add `bind_ip` param into forward plugin block
Example:
```
forward . 223.5.5.5 119.29.29.29 {
    bind_ip 192.168.1.2:0
}
```

6. Replace the coredns binary to your dns server

7. Restart the dns server

## What's my usage
```
. {
    hosts /etc/hosts {
        loop
        ttl 3600
        reload 10s
        fallthrough
    }

    forward . 127.0.0.1:1153 127.0.0.1:1253 127.0.0.1:1353 {
        prefer_udp
        policy round_robin
    }

    log
    errors
    # No cache here
}

.:1153 {
    forward . 223.5.5.5 119.29.29.29 {
        bind_ip 192.168.0.2:0
        prefer_udp
        policy sequential
    }

    # Cache here
    cache {
      success 10000 3600 1
      prefetch 500
      denial 0
    }
}

.:1253 {
    forward . 223.5.5.5 119.29.29.29 {
        bind_ip 192.168.1.2:0
        prefer_udp
        policy sequential
    }

    cache {
      success 10000 3600 1
      prefetch 500
      denial 0
    }
}

.:1353 {
    forward . 223.5.5.5 119.29.29.29 {
        bind_ip 192.168.2.2:0
        prefer_udp
        policy sequential
    }

    cache {
      success 10000 3600 1
      prefetch 500
      denial 0
    }
}
```
