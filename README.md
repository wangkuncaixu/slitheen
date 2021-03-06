#Slitheen is a decoy routing system for censorship resistance. The source code consists of two parts: the client code (to be distributed and run on the machines of users wishing to circumvent censorship), and the relay station code (to be run on a relay station deployed by an ISP in the middle of the network).

#If you have any questions about the code or installation, please contact Cecylia Bocovich <cbocovic@uwaterloo.ca>.


### INSTALLTION INSTRUCTIONS: ###

target=$1
if [ "$target" == "client" ];
then

#For the client:
#	- The client code consists of two parts: (1) an Overt User Simulator (OUS) that issues repeated requests to uncensored sites, and (2) a SOCKS proxy frontend that takes connection requests and data from the user's browser and feeds it to the OUS.
#
#0. Building this system requires the following dependencies:
	sudo apt-get install build-essential g++ flex bison gperf ruby perl \
	  libsqlite3-dev libfontconfig1-dev libicu-dev libfreetype6 libssl-dev \
	    libpng-dev libjpeg-dev python libx11-dev libxext-dev git

#1. Fetch all necessary git repositories:
        mkdir client
        cd client
	git clone git://git-crysp.uwaterloo.ca/slitheen
	git clone -b slitheen git://git-crysp.uwaterloo.ca/openssl
	git clone -b slitheen git://git-crysp.uwaterloo.ca/phantomjs

#2. Build our modified version of openssl
        parent_dir=`pwd`
	cd openssl
	./config --prefix=$parent_dir/sslout --openssldir=$parent_dir/sslout/openssl
	make
	make test
	make install
	cd ..

#3. Build phantomJS (Note: this takes a ridiculously long time)
	cd phantomjs
        sed -i "s#sslout#$parent_dir/sslout#g" build.py
	./build.py
	cp bin/phantomjs ../slitheen/client
        cd ..

#4. Build SOCKS proxy frontend
        cd slitheen/client
        make
        
#5. Run socks and exit to initialize OUS_out pipe
        ./socks
	
#6. Obtain public key from relay station save file as pubkey in client/ directory


elif [ "$target" == "relay" ];
then

#For the relay station:

#0. Install dependencies
	sudo apt-get install libpcap-dev libssl-dev git

#1. Fetch necessary git repository:
	git clone git://git-crysp.uwaterloo.ca/slitheen
	git clone -b slitheen git://git-crysp.uwaterloo.ca/openssl

#2. Build our modified version of openssl
        parent_dir=`pwd`
	cd openssl
	./config --prefix=$parent_dir/sslout --openssldir=$parent_dir/sslout/openssl
	make
	make test
	make install
        cp ../sslout/lib/*.a ../slitheen/relay_station
	cd ..

#3. Generate public/private key pair
	cd slitheen/telex-tag-v3
	make
	./genkeys
	cp privkey ../relay_station
	cd ..

#4. Update slitheen.h with correct MAC addresses for client-side and world-side interfaces
#    If using VM setup, postpone until VM step 10.

#5. Build slitheen proxy
	cd relay_station
	make

else 
    echo "Usage: ./INSTALL [client|relay]"

fi

### VIRTUAL MACHINE SETUP ###

#    If you wish to do a test run on two virtual machines on a single host, follow these instructions
#
#1. Download an Ubuntu 14.04 .iso (http://releases.ubuntu.com/14.04/)
#
#2. Create an Ubuntu14.04 virtual machine named "slitheen_client"
#
#3. Create an Ubuntu14.04 virtual machine named "slitheen_relay"
#
#4. Install the relay and client onto their respective machines following the
#    installation instructions below.
#
#5. Go to the network settings for "slitheen_client" and change Adapter 1 to be
#    attached to an internal network named "slitheen_net".
#
#6. In the network settings for "slitheen_relay", enable Adapter 2 and attach
#    it to the internal network "slitheen_net". Set promiscuous mode to "allow VMs".
#
#7. With "slitheen_relay" running, go to the network settings, select the adapter
#    connected to the internal network (eth1), and manually enter the following IPv4 settings:
#	Address: 192.168.3.1 Netmask:255.255.255.0 Gateway: 0.0.0.0
#
#8. With "slitheen_client" running, go to the network settings and manually enter
#    the following IPv4 settings:
#	Address: 192.168.3.2 Netmask:255.255.255.0 Gateway: 192.168.3.1
#
#9. Set the DNS server to match the one for eth0 on "slitheen_relay"
#
#10. In relay station code file slitheen.h, set macaddr to client's MAC address and compile
#
#11. Restart client and test connection
#	ping 192.168.3.1
#	ping 8.8.8.8


### RUN INSTRUCTIONS ###

#1. Run relay
#        ethtool -K [eth0] tso off
#        ethtool -K [eth0] gro off
#        ethtool -K [eth0] gso off
#        ethtool -K [eth1] tso off
#        ethtool -K [eth1] gro off
#        ethtool -K [eth1] gso off

#	./slitheen-proxy [interface to client] [interface to world]


#2. Run client
#        ./ous.sh
#        ./socks
#        Connect browser to localhost:1080


