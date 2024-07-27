#sd 

Ip 10.0.0.0
Subnet 255.255.255.0 

Meaning: First 3 bytes (24 bits) are used for the network and last byte for the device
Equivalent  = 10.0.0.0/24 (range from 10.0.0.0 to 10.0.0.255)

Notable network masks: 
- /8 => mask 255.0.0.0 
- /16 => mask 255.255.0.0 
- /24 => mask 255.255.255.0
- /32 => mask 255.255.255.255

Note: /1 => 128, /2 => 192, \3 => 224, \4 => 240, \5 => 248, \6 => 252, \7 => 254

Increasing mask by 1 => Divide nb of possible hosts by 2




