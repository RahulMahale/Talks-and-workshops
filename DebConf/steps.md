
 Detailed Document to be run on Raspbian to prepare it as a Wifi Router
 
steps for configuring Pi as Wifi Router

    sudo apt-get install hostapd udhcpd


edit /etc/udcpd.conf

    start 192.168.9.2 
    end 192.168.9.20
    interface wlan0 # The device uDHCP listens on.
    remaining yes
    opt dns 8.8.8.8 4.2.2.2 # The DNS servers client devices will use.
    opt subnet 255.255.255.0
    opt router 192.168.9.1 
    opt lease 864000 # 

Edit the file /etc/default/udhcpd and change the line:

     DHCPD_ENABLED="no"

to

    DHCPD_ENABLED="yes" #or comment it out


    sudo ifconfig wlan0 192.168.42.1

To set this up automatically on boot, edit the file /etc/network/interfaces and replace the line "iface wlan0 inet dhcp" to:

     iface wlan0 inet static
       address 192.168.9.1
       netmask 255.255.255.0

If the line "iface wlan0 inet dhcp" is not present, add the above lines to the bottom of the file.

Comment the following lines (they probably wont all be next to each other):

    allow-hotplug wlan0
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
    iface default inet manual



3. Configure HostAPD. You can create an open network, or a WPA-secured network. A secure network is recommended to prevent unauthorized use and tampering, but you can also create an open network. To create a WPA-secured network, edit the file /etc/hostapd/hostapd.conf (create it if it doesnt exist) and add the following lines


     interface=wlan0
     driver=nl80211
     ssid=My_AP
     hw_mode=g
     channel=6
     macaddr_acl=0
     auth_algs=1
     ignore_broadcast_ssid=0
     wpa=2
     wpa_passphrase=My_Passphrase
     wpa_key_mgmt=WPA-PSK
     wpa_pairwise=TKIP
     rsn_pairwise=CCMP

If you would like to create an open network, put the following text into /etc/hostapd/hostapd.conf:

     interface=wlan0
     ssid=My_AP
     hw_mode=g
     channel=6
     auth_algs=1
     wmm_enabled=0

Change ssid= and channel= to values of your choice. Note that anyone will be able to connect to your network, which is generally not a good idea. Also, some regions will hold an access point's owner responsible for any traffic that passes though an open wireless network, regardless of who actually caused that traffic.

Edit the file /etc/default/hostapd and change the line:
uncomment and chenge it to
   

     DAEMON_CONF="/etc/hostapd/hostapd.conf"


4. Configure NAT (Network Address Translation). NAT is a technique that allows several devices to use a single connection to the internet. Linux supports NAT using Netfilter (also known as iptables) and is fairly easy to set up. First, enable IP forwarding in the kernel


     sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

To set this up automatically on boot, edit the file /etc/sysctl.conf and add the following line to the bottom of the file:

     net.ipv4.ip_forward=1

Second, to enable NAT in the kernel, run the following commands:

     sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
     sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
     sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

These instructions don't give a good solution for rerouting https and for URLs referring to a page inside a domain, like abc.com/abc.htm. The user will see a 404 error. Your Pi is now NAT-ing. To make this permanent so you don't have to run the commands after each reboot, run the following command:

     sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

Now edit the file /etc/network/interfaces and add the following line to the bottom of the file:

     up iptables-restore < /etc/iptables.ipv4.nat

5. Fire it up! Run the following commands to start the access point


     sudo service hostapd start
     sudo service udhcpd start

Your Pi should now be hosting a wireless hotspot. To get the hotspot to start on boot, run these additional commands:

     sudo update-rc.d hostapd enable
     sudo update-rc.d udhcpd enable


