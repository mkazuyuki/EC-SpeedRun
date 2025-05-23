# EC-SpeedRun

## Configuring Disk


## Configuring NIC

```ps1
# Primary host

## Disabling IPv6
netsh interface ipv6 set state disabled

## Configuring IP address & Gateway
netsh interface ip set address name="Ethernet"   static 192.168.137.11 255.255.255.0 192.168.137.1
netsh interface ip set address name="Ethernet 2" static      10.0.0.11 255.0.0.0

## Configuring DNS
netsh interface ipv4 set dns name="Ethernet" source=static addr=192.168.137.1 register=primary

```

```ps1
# Secondary host

## Disabling IPv6
netsh interface ipv6 set state disabled

## Set IP address & Gateway
netsh interface ip set address name="Ethernet"   static 192.168.137.12 255.255.255.0 192.168.137.1
netsh interface ip set address name="Ethernet 2" static      10.0.0.12 255.0.0.0

## Configuring DNS
netsh interface ipv4 set dns name="Ethernet" source=static addr=192.168.137.1 register=primary
```
