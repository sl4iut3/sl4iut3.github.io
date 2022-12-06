
# RE 14 - VRF

## Introduction

idée reprise du document http://www.packetu.com/2012/07/12/vrfing-101-understing-vrf-basics/

## Topologie physique


```
                                      192.168.1.2 +----+
                              +-----------------  | R2 |
                              | vlan2       fa0/1 +----+
    fa0/1 +----+    trunk    S
    ----- | R1 |------------ W
      10. +----+  fa0/0      I
      23.       192.168.1.1  | 
      0.                     |        192.168.1.2 +----+
      88.                    +- ----------------  | R3 |
                                vlan3       fa0/1 +----+
```

## Config basique routeur R3         

après un *en*, *conf t*:

    int fa 0/1
    no shutdown
    ip addr 192.168.1.1 255.255.255.0


## Config moins basique routeur R2         

après un *en*, *conf t*:

    ip vrf red
    int fa 0/1
    no shutdown
    ip vrf forwarding red
    ip addr 192.168.1.1 255.255.255.0
    int fa 0/0 
    no shutdown
    ip addr 10.10.10.254 255.255.255.0
    
    ! config routeur ospf (fct pas bien....)
    router ospf vrf red
    network 192.168.1.0 0.0.0.255 area 1
    redistribute connected
    
## Config routeur R1         

après un *en*, *conf t*:

    ip vrf red
    ip vrf blue
    
    int fa 0/0
    no shutdown
    no ip addr
    int fa 0/0.1
    ip vrf forwarding red
    encap dot1q 2
    ip addr 192.168.1.1 255.255.255.0

    int fa 0/0.2
    ip vrf forwarding blue
    encap dot1q 3
    ip addr 192.168.1.1 255.255.255.0

plus NAT (classique) 

## Tests

### depuis R1

    ping vrf red 192.168.1.2    # ping sur R2
    ping vrf blue 192.168.1.2   # ping sur R3
    
ping sur internet fct (avant de placer l'interface ext en vrf red, me semble-t-il)

```
R1#ping 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R1#ping vrf red 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
R1#ping vrf blue 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
R1#ping 8.8.8.8             

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R1#ping vrf red 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R1#ping vrf red 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)


```

### depuis R3

    ping 192.168.1.1    # ping sur R1, sans connaissance de VRF
    
```
R3#ping 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
R3#ping 8.8.8.8    

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R3#ping 10.23.0.88

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.0.88, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#ip rou
R3(config)#ip route 0.0.0.0 0.0.0.0 192.168.1.1
R3(config)#exit
R3#ping 10.23.0.88
*Jan  1 01:41:50.255: %SYS-5-CONFIG_I: Configured from console by console

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.0.88, timeout is 2 seconds:
U.U.U
Success rate is 0 percent (0/5)
R3#ping 10.23.0.88

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.0.88, timeout is 2 seconds:
U.U.U
Success rate is 0 percent (0/5)

```

### depuis R2

    ping vrf red 192.168.1.1    # ping sur R1 en connaissant la VRF
    ping 192.168.1.1            # ping sur R1, sans VRF (ne fct pas, normal)

```
R2#ping 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R2#ping vrf red 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
R2#ping vrf red 10.23.0.88 

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.0.88, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
R2#ping vrf red 10.23.0.254

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.0.254, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R2#ping 10.23.0.88         

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.23.0.88, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

```
    
## config complètes

### config R2

    version 12.4
    service timestamps debug datetime msec
    service timestamps log datetime msec
    no service password-encryption
    !
    hostname R2
    !
    boot-start-marker
    boot-end-marker
    !
    !
    no aaa new-model
    memory-size iomem 5
    ip cef
    !
    ip vrf red
    !
    multilink bundle-name authenticated
    !
    !
    voice-card 0
    !
    archive   
    log config
    hidekeys    
    !
    interface FastEthernet0/0
    ip address 10.10.10.254 255.255.255.0
    duplex auto
     speed auto
    !
    interface FastEthernet0/1
    ip vrf forwarding red
    ip address 192.168.1.2 255.255.255.0
     duplex auto
     speed auto
    !
    router ospf 150 vrf red
    log-adjacency-changes
     redistribute connected subnets
    network 192.168.1.0 0.0.0.255 area 1
    !
    ip route vrf red 0.0.0.0 0.0.0.0 192.168.1.1
    !
    line con 0
    line aux 0
    line vty 0 4
    login   
    !
    scheduler allocate 20000 1000
    end

### config R3

```
version 12.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R3
!
boot-start-marker
boot-end-marker
!
!
no aaa new-model
memory-size iomem 5
ip cef
!
multilink bundle-name authenticated
!         
voice-card 0
!
archive
 log config
  hidekeys
!
interface FastEthernet0/0
 no ip address
 shutdown
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 192.168.1.2 255.255.255.0
 duplex auto
 speed auto
!
line con 0
line aux 0
line vty 0 4
 login
!
scheduler allocate 20000 1000
end
```

### config R1

```
version 12.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!
boot-start-marker
boot-end-marker
!
!
no aaa new-model
memory-size iomem 5
ip cef
!
!
!
!
ip vrf TOTO
!
ip vrf blue
!
ip vrf red
!
multilink bundle-name authenticated
!
!
voice-card 0
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!         
!
!
!
archive
 log config
  hidekeys
!
!
!
!
!
!
interface Loopback10
 ip vrf forwarding red
 ip address 192.168.25.1 255.255.255.0
!
interface FastEthernet0/0
 no ip address
 ip nat inside
 ip virtual-reassembly
 duplex auto
 speed auto
!         
interface FastEthernet0/0.1
 encapsulation dot1Q 2
 ip vrf forwarding red
 ip address 192.168.1.1 255.255.255.0
!
interface FastEthernet0/0.2
 encapsulation dot1Q 3
 ip vrf forwarding blue
 ip address 192.168.1.1 255.255.255.0
!
interface FastEthernet0/1
 ip vrf forwarding red
 ip address 10.23.0.88 255.255.255.0
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface Serial0/1/0
 no ip address
 shutdown
 clock rate 125000
!         
interface Serial0/1/1
 no ip address
 shutdown
 clock rate 125000
!
router ospf 200 vrf red
 log-adjacency-changes
 network 192.168.1.0 0.0.0.255 area 1
!
ip route 0.0.0.0 0.0.0.0 10.23.0.254
!
!
no ip http server
no ip http secure-server
ip nat source list 100 interface FastEthernet0/1
!
access-list 100 permit ip 192.168.1.0 0.0.0.255 any
!
!
!
control-plane
!
!         
!
!
!
!
!
!
!
line con 0
line aux 0
line vty 0 4
 login
!
scheduler allocate 20000 1000
end

```

