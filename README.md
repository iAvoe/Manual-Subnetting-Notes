### Subnetting relates to network securiy and stability, this is not an abstract.

## Step 1 - Preparations

### Create "Subnet and wildcard subnet baked.txt" from content below:

    Subnet Mask & Wlcd Mask Slash Hosts (minus 2):
    Broadcast .255 and Router .1 takes that 2 off
    
    Subnet Mask     Wlcd mask       CIDR    Binary   Hosts available
    255.255.255.255	0.0.0.0		/32     11111111
    255.255.255.254	0.0.0.1		/31	11111110
    255.255.255.252	0.0.0.3		/30	11111100 = 2  hosts
    255.255.255.248	0.0.0.7		/29	11111000 = 6  hosts
    255.255.255.240	0.0.0.15	/28	11110000 = 14 hosts
    255.255.255.224	0.0.0.31	/27 	11100000 = 30 hosts
    255.255.255.192	0.0.0.63	/26 	11000000 = 62 hosts
    255.255.255.128	0.0.0.127	/25	10000000 = 126hosts
    255.255.255.0	0.0.0.255	/24	00000000 = 254hosts (256-2)
    
    255.255.254.0	0.0.1.255	/23     11111110 00000000 = 510hosts (512-2)
    255.255.252.0	0.0.3.255	/22
    255.255.248.0	0.0.7.255	/21
    255.255.240.0	0.0.15.255	/20              = 4094 hosts (4096-2)
    255.255.224.0	0.0.31.255	/19
    255.255.192.0	0.0.63.255	/18
    255.255.128.0	0.0.127.255	/17
    
    255.255.0.0	0.0.255.255	/16
    255.254.0.0	0.1.255.255	/15
    255.252.0.0	0.3.255.255	/14
    255.248.0.0	0.7.255.255	/13
    255.240.0.0	0.15.255.255	/12
    255.224.0.0	0.31.255.255	/11
    255.192.0.0	0.63.255.255	/10
    255.128.0.0	0.127.255.255	/9
    
    255.0.0.0	0.255.255.255	/8
    254.0.0.0	1.255.255.255	/7
    252.0.0.0	3.255.255.255	/6
    248.0.0.0	7.255.255.255	/5
    240.0.0.0	15.255.255.255	/4
    224.0.0.0	31.255.255.255	/3
    192.0.0.0	63.255.255.255	/2
    128.0.0.0	127.255.255.255	/1
    000.0.0.0	255.255.255.255	/0

    Find the minimal subnet to cover the maximum amount of hosts supported here
    Static Routing should be called as reverse-gateway, remember this.

## Step 2 - Find where to compute

### Example - 154.56.141.11/20

     1. Convert CIDR/Subnet/Wildcard bits to binary
     - Find where does a CIDR of /20 corresponds to the file above
     - According from the 'baked' subnet map, CIDR /20 corresponds 255.255.240.0 / 0.0.15.255
     - We get:
       - 2 × 255 decimal bits
       - 1 × 240 decimal bit
       - 1 × 000 decimal bit
     - 240, according to the Binary column in the 'baked' map, corresponds to 1111000
     - The rest parts are easy, we get:
       - 2 × 11111111 binary bits
       - 1 × 11110000 binary bit
       - 1 × 00000000 binary bit
     - Now we have a subnet converted from decimal to binary as 11111111.11111111.11110000.00000000
     
     2. Find which the device/node's bits
     - Convert 1-to-N, 0-to-D, the binary subnet bits becomes NNNNNNNN.NNNNNNNN.NNNNDDDD.DDDDDDDD
     - N stands for network or Routers' external address, most of them are available to be assigned by your ISP
     - D stands for Device/Nodes' address range, all of them are available for devices, or the entire DHCP pool
     - We only need to compute the address range with both N, D shows up
     
     3. The 'D' bits are found
     - There could be 0b_0000,00000000 to 0b_1111,11111111 devices, minus Router's internal & broadcast addresses (2) available

**Note:** Binary addition:

    0d256 = 0b_1,0000,0000 = 0b_1000,0000+0b1000,0000 = 0d 128+128
    0d128 = 0b___1000,0000 = 0b_0100,0000+0b0100,0000 = 0d 64+64

## Step 3 - Get binary

### Example - 154.56.141.11/20

     - We only compute where 'N' bits and 'D' bits occurs in the same range, which in this case, the '141' portion
     - Convert "141" to binary
       - 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255
     - 141 is larger than 128, subtract it, and we have 128 and 13
       - 128 + 00 + 00 + 00 + ............. = 141
     - 8 + 4 = 12, which is smaller than 13, subtract it.
       - 128 + 00 + 00 + 00 + 8 + 4 + ..... = 141
     - Only 1 is left, which is:
       - 128 + 00 + 00 + 00 + 8 + 4 + 0 + 1 = 141
       - 0b1,  0,   0,   0,   1,  1,  0,  1 = 0d_141
     - We found that this corresponds to 1, -, -, -, 1, 1, -, 1, or 10001101
     - The rest 'NNNNNNNN' or 'DDDDDDDD' 154, 56 and 11 as 0d154, 0d56 & 0d11
     - We get 0d_154.0d_56.0b_10001101.0d_11

## Step 4 - Filtering

Subnet mask cuts off all the devices bit, leaving network bits; Which means whereever "D" exists, all "1"s becomes 0 in the ip address

    Before---------------------------    0d154.----0d56.10001101.----0d11
    Filtering------------------------ NNNNNNNN.NNNNNNNN.NNNNDDDD.DDDDDDDD
    Where D bits exists, convert to 0    0d154.----0d56.10000000.00000000
    
    Back-to-decimal: 154.56.128.0

And we are done.

-----

# Deal with actual senarios with ease

### Example 1: 172.16.28.252, wlcd 0.0.15.255

     1. mask is 255.255.240.0, CIDR is /20, promask is NNNNNNNN.NNNNNNNN.NNNNDDDD.DDDDDDDD
     2. 0d172.0d16.20+8.0d252
     3. 0d172.0d16.16+8+4.0d252
     4. 0d172.0d16.16+8+4.0d252, where 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255
     5. 0d172.0d16.00011100.0d252
     6. DDDD.DDDDDDDD, filter 12bits
     7. 172.16.0b00010000.0
     8. 172.16.16.0 is the network address, 172.16.16.1 is the 1st available address
     9. 172.16.00011111.11111111 is the broadcast address
     10. 00011111 is 16 plus (the rest that always makes up to 16), which is 172.16.32.255
 
### Example 2: 192.168.20.24/29, configure web server to last usable

     1. 255.255.255.248	0.0.0.7		/29	11111000 = 6  hosts, 8N.8N.8N.NNNNNDDD
     2. 0d192.0d168.0d20.16+8, where 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255
     3. 192.168.20.00011000, strip there D exists
     4. 192.168.20.00011000 is the network address, which is 192.168.20.24
     5. 192.168.20.00011001 is the 1st usable address, which is 192.168.20.25
     6. 192.168.20.00011111 is the broadcast address, which is 192.168.20.31
     7. The web server's IP should be 192.168.20.30
 
### Example 3 (hard): Setup 8 LANs starts from 192.168.2.1 with normal subnet, each contain 5~26 hosts
 
    1. max(8,26)=26
    2. 255.255.255.240	0.0.0.15	/28	    11110000 = 14 hosts
       255.255.255.224	0.0.0.31	/27 	11100000 = 30 hosts (this the minimal subnet size available)
    3. 8N.8N.8N.NNNDDDDD
    4. LAN1: 198.168.2.000 00000 ~ 198.168.2.000 11111
       LAN2: 198.168.2.001 00000 ~ 198.168.2.001 11111
       LAN3: 198.168.2.010 00000 ~ 198.168.2.010 11111
       LAN4: 198.168.2.011 00000 ~ 198.168.2.011 11111
       LAN5: 198.168.2.100 00000 ~ 198.168.2.100 11111
       LAN6: 198.168.2.101 00000 ~ 198.168.2.101 11111
       LAN7: 198.168.2.110 00000 ~ 198.168.2.110 11111
       LAN8: 198.168.2.111 00000 ~ 198.168.2.111 11111

    5. LAN1 usable: 198.168.2.01 ~ 198.168.2.32
       LAN2 usable: 198.168.2.33 ~ 198.168.2.64
       LAN3 usable: 198.168.2.65 ~ 198.168.2.128
       ...

### Example 4 (very hard): Setup a variable subnet mask (VLSM) for the following dpts in a company, choose your own IP address:

    25	    IT/Mgmt.
    150	    Marketing
    200	    Production
    60 	    Accounting
    24	    HR & Payroll
    12	    Research

    1. max(20, 150, 200,60,24, 12)=200
    2. 255.255.255.0	0.0.0.255	/24	    00000000 = 254hosts (256-2) fits 200 people
    3. choosing 10.0.1.0 as network IP
    4. End user size in order: Production > Marketing > Accounting 60 > HR & payoll 24 > IT 25 > Research 12
    5. The rest: 64+32+32+6=134
    
    6. 255.255.255.248	0.0.0.7		/29	11111000 = 6  hosts
       255.255.255.240	0.0.0.15	/28	11110000 = 14 hosts
       255.255.255.224	0.0.0.31	/27 	11100000 = 30 hosts
       255.255.255.192	0.0.0.63	/26 	11000000 = 62 hosts
       
    7. Department                           Network Addr    First Avail.    Last Avail.     VLSM used
       Production	CIDR: 24 (254hosts)	= 10.0.1.0/24   = 10.0.1.1      = 10.0.1.255    0
       Marketing	CIDR: 24 (254hosts)	= 10.0.2.0/24   = 10.0.2.1      = 10.0.2.255    0
       Accounting	CIDR: 26 (62hosts = 62) = 10.0.3.0/26   = 10.0.3.1      = 10.0.3.64     1
       HR & Payroll	CIDR: 26 (24hosts < 62) = 10.0.3.0/26   = 10.0.3.65     = 10.0.3.128    1
       IT           CIDR: 26 (25hosts < 62) = 10.0.3.0/26   = 10.0.3.129    = 10.0.3.192    1
       Research     CIDR: 26 (6hosts  < 62) = 10.0.3.0/26   = 10.0.3.193    = 10.0.3.255    1

### Example 5 - (PC DIY): hook up an Eth printer to your computer directly via ethernet

You've found a good deal on eBay where companies/schools dumps enterprise printers which are infinitely better than your 2000era DeskJets (no offence)
 - You want to put this printer in the other side of your room because of it's size, and a USB cable is too short for this distance
 - You don't want other tenants sharing this network to see and ask you to print, scan for them
 - The printer has 1 Eth, 1 USB type B port available

Choosing USB over Eth is more expensive and the signal quality / clearity would be horrible:
 - 1 spool of Cat5 cable
 - 2 USB to Eth adaptors
 - 1 USB-A to USB-B adaptor

Choosing Eth is cheaper and more elegant, and less likely to pick up EMI noises:
 - 1 spool of Cat5 cable
 - 1 PCIE to Eth adaptor (e.g., Realtek 8111)

Therefore you configures the ip address using the same principle, creating a private network with something like 10.0.0.0:
#
    01	    A pcie to Eth adaptor
    01	    A network printer

    1. max 1+1=2 devices
    2. 255.255.255.252	0.0.0.3	    /30		11111100 = 2 hosts
    3. Department		Network Addr	First Avail.	 Last Avail.	VLSM used
       Private network	= 10.0.0.0/30	= 10.0.0.1	 = 10.0.0.2	0

Finally, manually set the network printer with:
 - IP address:  10.0.0.2
 - Subnet mask: 255.255.255.252
 - Def Gateway: 10.0.0.1

And the PCIE NIC on your PC with:
 - IP address:  10.0.0.1
 - Subnet mask: 255.255.255.252
 - Def Gateway: {home router's IP address}

If you changes your mind and wanted to share the network printer later, add a static route with:
 - network 10.0.0.0 with subnet mask 255.255.255.252, via {your PC's static IP address}
