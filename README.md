# Lighweight Network EMulator

Lighweight Network EMulator simplifies integration testing of network applications on Linux. It is implemented as a collection of easy-to-use 
Bash scripts on top of Linux network namespaces and Netem.

## How it works 
Lighweight Network EMulator creates a number of Linux network namespaces and places them in a star topology around the host machine. 
The network namespaces are named blue1, blue2, ... , blueN. Each network namespace is connected to the host with the help of a pair of virtual ethernet devices. For network namespace blueN, the virtual ethernet devices are named vethrealN and vethvirtualN, respectively. The device vethvirtualN is only visible
from the namespace blueN, and the device vethrealN is visible from the host. The connection between the vethrealN and vethvirtualN devices is traffic-shaped using Netem and the tc command to introduce emulated latency, and limits for the uplink and downlink bandwidth. The emulated network uses the 10.240.0.0/12 network range. In case of emulated  networks with less than 255 namespaces, the IP adderess of the device vethrealN is 10.240.N.1 and the IP address of device vethvirtualN 10.240.N.2, respectively. All routing towards the other network namespaces and the Internet goes through the host machine.






```
                                               Internet 
                                                  |                                       
  _____________________                           |                                _____________________
 |                     |                          |                               |                     |
 |  vethvirtual1       |-------- vethreal1 ----- host ----- vethreal2 ------------|  vethvirtual2       |  
 |  10.240.1.2         |         10.240.1.1       |         10.240.2.1            |  10.240.2.2         |
 |  namespace blue1    |                          |                               |  namespace blue2    |
 |_____________________|                          |                               |_____________________|   
                                              vethrealN
											  10.240.N.1  
                                                  |
                                                  |  
                                                  |
										___________________________ 
									   |                           |
									   |  vethvirtualN             |
									   |  10.240.N.2               |
									   |  namespace blueN          |
									   |                           | 
									   |___________________________|   


```

### Prerequisites

* A Linux host with root privilleges (use a dedicated virtual machine to stay safe)
* The following Debian packages (or equivivalent):

```
sudo apt-get install iproute
```

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details


