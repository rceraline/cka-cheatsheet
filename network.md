# List of connection

```
netstat -natulp
```

## Create a bridge network

```
ip link add v-net-0 type bridge

ip link set dev v-net-0 up

```

## Set the IP address for the bridge interface

```
ip addr add 10.244.1.1/24 dev v-net-0
```

## Route traffic

```
ip route add 10.244.2.2 via 192.168.1.12
```
