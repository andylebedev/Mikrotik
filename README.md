# Mikrotik
Mikrotik config for receiving internet over tethering wi-fi

# 2025-07-04 16:41:44 by RouterOS 7.19.2
# software id = ELHR-FXHG
#
# model = C52iG-5HaxD2HaxD
# serial number = HGR0ABDX329
/interface bridge
add admin-mac=F4:1E:57:04:5F:38 auto-mac=no comment=defconf name=bridge
/interface list
add comment=defconf name=WAN
add comment=defconf name=LAN
/interface wifi channel
add band=2ghz-ax disabled=no frequency=2412,2437,2462 name=channel2 width=20mhz
add band=5ghz-ax disabled=no name=channel5 skip-dfs-channels=all
/interface wifi security
add authentication-types=wpa2-psk,wpa3-psk connect-priority=0 disabled=no name=Phone-connect
add authentication-types=wpa2-psk,wpa3-psk connect-priority=0 disabled=no ft=yes group-key-update=1h name=home-security
/interface wifi configuration
add channel=channel2 country=Canada disabled=no mode=ap name=home2 security=home-security security.connect-priority=0 ssid=home
add disabled=no mode=station-pseudobridge name=Phone-connect-config security=Phone-connect security.connect-priority=0 ssid=Phone
add channel=channel5 country=Canada disabled=no mode=ap name=home5 security=home-security security.connect-priority=0 ssid=home
/interface wifi
set [ find default-name=wifi2 ] configuration=home2 configuration.mode=ap disabled=no security.connect-priority=0 .ft=yes .ft-over-ds=yes
set [ find default-name=wifi1 ] configuration=Phone-connect-config configuration.mode=station .ssid=Phone disabled=no name=wifi5 security=Phone-connect security.connect-priority=0 .ft=yes .ft-over-ds=yes
/ip pool
add name=default-dhcp ranges=10.0.0.10-10.0.0.250
add name=iot ranges=10.0.13.10-10.0.13.250
/ip dhcp-server
add address-pool=default-dhcp interface=bridge name=defconf
/ipv6 pool
add name=ipv6-pool prefix=fd00::/64 prefix-length=64
/user group
add name=homeassistant policy=read,test,api,!local,!telnet,!ssh,!ftp,!reboot,!write,!policy,!winbox,!password,!web,!sniff,!sensitive,!romon,!rest-api
/interface bridge port
add bridge=bridge comment=defconf interface=ether2
add bridge=bridge comment=defconf interface=ether3
add bridge=bridge comment=defconf interface=ether4
add bridge=bridge comment=defconf interface=ether5
add bridge=bridge comment=defconf interface=wifi2
/ip neighbor discovery-settings
set discover-interface-list=LAN
/interface detect-internet
set detect-interface-list=all
/interface list member
add comment=defconf interface=bridge list=LAN
add comment=defconf interface=wifi5 list=WAN
add interface=ether1 list=WAN
/interface ovpn-server server
add mac-address=FE:68:DC:77:FD:19 name=ovpn-server1
/interface wifi capsman
set enabled=yes package-path=/packages require-peer-certificate=no upgrade-policy=suggest-same-version
/interface wifi provisioning
add action=create-dynamic-enabled disabled=no master-configuration=home2 supported-bands=2ghz-ax
add action=create-dynamic-enabled disabled=no master-configuration=home5 supported-bands=5ghz-ax
/ip address
add address=10.0.0.1/24 comment=defconf interface=bridge network=10.0.0.0
/ip dhcp-client
# Interface not active
add comment=defconf interface=ether1
add interface=wifi5
/ip dhcp-server lease
add address=10.0.0.215 client-id=ff:3e:f0:48:b8:0:3:0:1:ec:4d:3e:f0:48:b8 mac-address=EC:4D:3E:F0:48:B8 server=defconf
add address=10.0.0.217 client-id=1:44:4f:8e:a1:a2:de mac-address=44:4F:8E:A1:A2:DE server=defconf
add address=10.0.0.218 client-id=1:44:4f:8e:a1:ee:2c mac-address=44:4F:8E:A1:EE:2C server=defconf
add address=10.0.0.219 client-id=1:44:4f:8e:9b:ca:f2 mac-address=44:4F:8E:9B:CA:F2 server=defconf
add address=10.0.0.200 client-id=1:b8:27:eb:5e:ba:93 mac-address=B8:27:EB:5E:BA:93 server=defconf
/ip dhcp-server network
add address=10.0.0.0/24 comment=defconf dns-server=10.0.0.1 gateway=10.0.0.1 netmask=24
/ip dns
set allow-remote-requests=yes cache-size=4096KiB servers=8.8.8.8
/ip dns adlist
add ssl-verify=no url=https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
/ip dns static
add address=10.0.0.1 comment=defconf name=router.lan type=A
add address=10.0.0.200 name=homeassistant.local type=A
/ip firewall address-list
add address=0.0.0.0/8 comment="defconf: RFC6890" list=no_forward_ipv4
add address=169.254.0.0/16 comment="defconf: RFC6890" list=no_forward_ipv4
add address=224.0.0.0/4 comment="defconf: multicast" list=no_forward_ipv4
add address=255.255.255.255 comment="defconf: RFC6890" list=no_forward_ipv4
/ip firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=input comment="defconf: drop invalid" connection-state=invalid
add action=accept chain=input comment="defconf: accept ICMP" protocol=icmp
add action=accept chain=input comment="defconf: accept to local loopback (for CAPsMAN)" dst-address=127.0.0.1
add action=drop chain=input comment="defconf: drop all not coming from LAN" in-interface-list=!LAN
add action=drop chain=input dst-port=53 in-interface-list=WAN protocol=udp
add action=drop chain=input dst-port=53 in-interface-list=WAN protocol=tcp
add action=accept chain=forward comment="defconf: accept in ipsec policy" ipsec-policy=in,ipsec
add action=accept chain=forward comment="defconf: accept out ipsec policy" ipsec-policy=out,ipsec
add action=fasttrack-connection chain=forward comment="defconf: fasttrack" connection-state=established,related hw-offload=yes
add action=accept chain=forward comment="defconf: accept established,related, untracked" connection-state=established,related,untracked
add action=drop chain=forward comment="defconf: drop invalid" connection-state=invalid
add action=drop chain=forward comment="defconf: drop all from WAN not DSTNATed" connection-nat-state=!dstnat connection-state=new in-interface-list=WAN
/ip firewall mangle
add action=change-ttl chain=postrouting new-ttl=set:65 out-interface=wifi5
/ip firewall nat
add action=masquerade chain=srcnat comment="defconf: masquerade" ipsec-policy=out,none out-interface-list=WAN
/ip ipsec profile
set [ find default=yes ] dpd-interval=2m dpd-maximum-failures=5
/ipv6 firewall address-list
add address=::/128 comment="defconf: unspecified address" list=bad_ipv6
add address=::1/128 comment="defconf: lo" list=bad_ipv6
add address=fec0::/10 comment="defconf: site-local" list=bad_ipv6
add address=::ffff:0.0.0.0/96 comment="defconf: ipv4-mapped" list=bad_ipv6
add address=::/96 comment="defconf: ipv4 compat" list=bad_ipv6
add address=100::/64 comment="defconf: discard only " list=bad_ipv6
add address=2001:db8::/32 comment="defconf: documentation" list=bad_ipv6
add address=2001:10::/28 comment="defconf: ORCHID" list=bad_ipv6
add address=3ffe::/16 comment="defconf: 6bone" list=bad_ipv6
/ipv6 firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=input comment="defconf: drop invalid" connection-state=invalid
add action=accept chain=input comment="defconf: accept ICMPv6" protocol=icmpv6
add action=accept chain=input comment="defconf: accept UDP traceroute" dst-port=33434-33534 protocol=udp
add action=accept chain=input comment="defconf: accept DHCPv6-Client prefix delegation." dst-port=546 protocol=udp src-address=fe80::/10
add action=accept chain=input comment="defconf: accept IKE" dst-port=500,4500 protocol=udp
add action=accept chain=input comment="defconf: accept ipsec AH" protocol=ipsec-ah
add action=accept chain=input comment="defconf: accept ipsec ESP" protocol=ipsec-esp
add action=accept chain=input comment="defconf: accept all that matches ipsec policy" ipsec-policy=in,ipsec
add action=drop chain=input comment="defconf: drop everything else not coming from LAN" in-interface-list=!LAN
add action=accept chain=forward comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=forward comment="defconf: drop invalid" connection-state=invalid
add action=drop chain=forward comment="defconf: drop packets with bad src ipv6" src-address-list=bad_ipv6
add action=drop chain=forward comment="defconf: drop packets with bad dst ipv6" dst-address-list=bad_ipv6
add action=drop chain=forward comment="defconf: rfc4890 drop hop-limit=1" hop-limit=equal:1 protocol=icmpv6
add action=accept chain=forward comment="defconf: accept ICMPv6" protocol=icmpv6
add action=accept chain=forward comment="defconf: accept HIP" protocol=139
add action=accept chain=forward comment="defconf: accept IKE" dst-port=500,4500 protocol=udp
add action=accept chain=forward comment="defconf: accept ipsec AH" protocol=ipsec-ah
add action=accept chain=forward comment="defconf: accept ipsec ESP" protocol=ipsec-esp
add action=accept chain=forward comment="defconf: accept all that matches ipsec policy" ipsec-policy=in,ipsec
add action=drop chain=forward comment="defconf: drop everything else not coming from LAN" in-interface-list=!LAN
/system clock
set time-zone-name=America/Toronto
/system scheduler
add interval=2d name=updater on-event="/system package update\
    \ncheck-for-updates once\
    \n:delay 5s;\
    \n:if ([get status] = \"New version is available\") do={install}" policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon start-date=2025-06-13 start-time=01:23:45
/tool mac-server
set allowed-interface-list=LAN
/tool mac-server mac-winbox
set allowed-interface-list=LAN
set show-at-login=no
/tool mac-server
set allowed-interface-list=LAN
/tool mac-server mac-winbox
set allowed-interface-list=LAN
