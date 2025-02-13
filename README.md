# Network Namespace Routing with Bridges

This assignement sets up a **virtual network topology** using Linux network namespaces, bridges, and routing. It enables communication between two namespaces (`ns1` and `ns2`) through a **router namespace (`router-ns`)**.

## Network Topology

![Topology](https://github.com/SaikatGeek/DevOps/blob/main/topology.jpg?raw=true)

## Steps to Set Up the Network

### 1️. Create Network Bridges
Create br0 and br1 bridges and up these bridges
```sh
sudo ip link add br0 type bridge
sudo ip link add br1 type bridge
sudo ip link set br0 up
sudo ip link set br1 up
```
#### Verify
```sh
ip link show br0
ip link show br1
```
### 2. Create Network Namespaces
Create ns1, ns2 and router-ns namespaces
```sh
sudo ip netns add ns1
sudo ip netns add ns2
sudo ip netns add router-ns
```
#### Verify
```sh
sudo ip netns list
```
### 3. Create Virtual Interfaces and Connections
```sh
sudo ip link add veth-ns1 type veth peer name veth-br0
sudo ip link add veth-r-ns1 type veth peer name veth-br0-ns1
sudo ip link add veth-ns2 type veth peer name veth-br1
sudo ip link add veth-r-ns2 type veth peer name veth-br1-ns2

sudo ip link set veth-ns1 netns ns1
sudo ip link set veth-ns2 netns ns2
sudo ip link set veth-r-ns1 netns router-ns
sudo ip link set veth-r-ns2 netns router-ns

sudo ip link set veth-br0 master br0
sudo ip link set veth-br0-ns1 master br0
sudo ip link set veth-br1 master br1
sudo ip link set veth-br1-ns2 master br1

sudo ip link set veth-br0 up
sudo ip link set veth-br0-ns1 up
sudo ip link set veth-br1 up
sudo ip link set veth-br1-ns2 up
```
#### Verify
```sh
sudo ip netns exec ns1 ip link show
sudo ip netns exec ns2 ip link show
sudo ip netns exec router-ns ip link show
```
### 4. Configure IP Addresses
Assign IP addresses to interfaces in namespaces and bridges

| Namespace   | Interface   | IP Address     | Subnet          |
|-------------|-------------|----------------|-----------------|
| ns1         | veth-ns1    | 10.11.0.10/24  | 10.11.0.0/24   |
| ns2         | veth-ns2    | 10.12.0.10/24  | 10.12.0.0/24   |
| router-ns   | veth-r-ns1  | 10.11.0.1/24   | 10.11.0.0/24   |
| router-ns   | veth-r-ns2  | 10.12.0.1/24   | 10.12.0.0/24  |

```sh
sudo ip netns exec ns1 ip addr add 10.11.0.10/24 dev veth-ns1
sudo ip netns exec ns2 ip addr add 10.12.0.10/24 dev veth-ns2

sudo ip netns exec router-ns ip addr add 10.11.0.1/24 dev veth-r-ns1
sudo ip netns exec router-ns ip addr add 10.12.0.1/24 dev veth-r-ns2
```
#### Verify
Check IP address assignment

```sh
sudo ip netns exec ns1 ip addr show
sudo ip netns exec ns2 ip addr show
sudo ip netns exec router-ns ip addr show
```
### 5. Set Up Routing
Configure routing within namespaces and enable IP forwarding

```sh
sudo ip netns exec ns1 ip route add default via 10.11.0.1
sudo ip netns exec ns2 ip route add default via 10.12.0.1
```
#### Enable Ip forwarding:
Allow packet forwarding process

```sh
sudo sysctl -w net.ipv4.ip_forward=1
sudo ip netns exec router-ns sysctl -w net.ipv4.ip_forward=1
```
#### Verify
```sh
sudo ip netns exec ns1 ip route show
sudo ip netns exec ns2 ip route show
sudo ip netns exec router-ns ip route show
```
### 6. Test Connectivity
Ping between namespaces by the router

Ping:
```sh
sudo ip netns exec ns1 ping -c 2 10.11.0.1
sudo ip netns exec router-ns ping -c 2 10.11.0.10
sudo ip netns exec ns2 ping -c 2 10.12.0.1
sudo ip netns exec router-ns ping -c 2 10.12.0.10

# Test ns1 to ns2 (via router)
sudo ip netns exec ns1 ping -c 2 10.12.0.10
sudo ip netns exec ns2 ping -c 2 10.11.0.10
```
### 7. Cleanup
Delete namespaces and bridges

```sh
sudo ip netns del ns1
sudo ip netns del ns2
sudo ip netns del router-ns
sudo ip link del br0
sudo ip link del br1
