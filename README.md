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
  ___________________                        |                         ___________________
 |                   |                       |                        |                   |
 |  vethvirtual1     |----- vethreal1 ----- host ----- vethreal2 -----|  vethvirtual2     |  
 |  10.240.1.2       |      10.240.1.1       |         10.240.2.1     |  10.240.2.2       |
 |  namespace blue1  |                       |                        |  namespace blue2  |
 |___________________|                       |                        |___________________|   
                                         vethrealN
                                         10.240.N.1  
                                             |
                                             |  
                                             |
                                     ___________________ 
                                    |                   |
                                    |  vethvirtualN     |
                                    |  10.240.N.2       |
                                    |  namespace blueN  | 
                                    |___________________|   


```

## Prerequisites

* A Linux host with root privilleges (use a dedicated virtual machine to stay safe)
* The following Debian packages (or equivivalent):

```
sudo apt-get install iproute
```

## Installing

```
sudo npm install -g @streamr/lnem
```

## Uninstalling

```
sudo npm uninstall -g @streamr/lnem
```

## Usage examples

## Measuring emulated network using ping and netperf

Install netperf 

```
sudo apt-get install netperf
```

Create a network namespace for the netperf server with download bandwidth limit of 1000 kbit/s, upload bandwidth limit of 2000 kbit/s, and latency of 10 ms.

```
sudo lnem-up --download 1000 --upload 2000 --latency 10 --num 1
```

Create a network namespace for the netperf client with no limits. Notice that we give the offset parameter
to start creating the network namespaces starting from namespace blue2, since namespace blue1 is already in use by the server.

```
sudo lnem-up --num 1 --offset 2
```

Measure the RTT between the blue1 and blue2 etwork namespaces using ping

```
sudo ip netns exec blue2 ping 10.240.1.2
```

Start the netperf server in the blue1 namespace

```
sudo ip netns exec blue1 netserver  &
```


Run the netperf client in the blue2 namespace, and instruct it to connect to the server running in namespace blue2. The result should
be close to 1 Mibit/s (the download bandwidth limit we set to the server's namespace)

```
sudo ip netns exec blue2 netperf -H 10.240.1.2 
```

To clean up, kill netperf server, and delete the two namespaces.

```
sudo pkill netserver
sudo lnem-down -n 2
```


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details


