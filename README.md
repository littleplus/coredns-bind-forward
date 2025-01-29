# coredns-bind-forward
本补丁主要用于对Coredns内置的forward插件添加绑定IP出口功能

[README in English?](https://github.com/littleplus/coredns-bind-forward/blob/main/README.en.md)

## 用途
本patch主要是提供给拥有多ISP网络接入的情况，例如电信、联通、移动三线接入，由于各种服务提供商（如百度、bilibili等）会提供基于ISP的DNS解析优化，我们需要使DNS通过指定的网络接口进行查询

## 如何食用
1. Clone coredns 源代码，checkout 到 commit id `51e11f166ef6c247a78e9e15468647c593b79b9f` 或者 tag `v1.12.0`
```
git clone https://github.com/coredns/coredns.git
cd coredns
git checkout tags/v1.12.0
```

2. 将patch放置到coredns源代码目录下
```
wget -q https://github.com/littleplus/coredns-bind-forward/raw/refs/heads/main/51e11f166ef6c247a78e9e15468647c593b79b9f.patch
```

3. 应用patch
```
git apply 
```

4. Build coredns
```
make
```

5. 编辑 coredns 配置文件, 添加 `bind_ip` 参数到 forward 插件的配置块
如:
```
forward . 223.5.5.5 119.29.29.29 {
    bind_ip 192.168.1.2:0
}
```

6. 替换编译的 coredns 二进制文件到你的服务器上，**记得备份原来的**

7. 重启 coredns

## 我的配置
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
