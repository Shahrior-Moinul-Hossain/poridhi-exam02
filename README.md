# poridhi-exam02
Poridhi Exam -Module 02 Docker Networking
<br>
Author-Shahrior Moinul Hossain


Task: Make two network namespace using 'red' and 'green' names. Connect tem with a bridge and check connectivity.

# At first, Update and Upgrade systems
	$ sudo apt update && apt upgrade

# Install necessary package
	$ sudo apt install net-tools iproute2 iputils-ping iptables -y

# Add Red Namespace
	$ sudo ip netns add red

# Add Green Namespace
	$ sudo ip netns add green

# Add Bridge Interface br0, according to task requirement 
	$ sudo ip link add br0 type bridge

# Assign IP address to the newly added bridge interface
	$ sudo ip addr add 172.16.10.10/24 dev br0
 
# UP the Bridge Interface
    	$ sudo ip link set br0 up

# Create a virtual link to add Red namespace and Bridge Interface
    $  sudo ip link add veth-red type veth peer name veth-br0-red

# Add one end of the virtual link to the Red namespace
	   $ sudo ip link set veth-red netns red

# Add another link of the virtual link to the Bridge Interface
	   $  sudo ip link set veth-br0-red master br0

# Up the bridge end of the newly created virtual cable. 
	   $  sudo ip link set veth-br0-red up

# Create a virtual link to add Green namespace and Bridge Interface
	   $ sudo ip link add veth-green type veth peer name veth-br0-green

# Add one end of the virtual link to the Green namespace
	   $ sudo ip link set veth-green netns green

# Add another link of the virtual link to the Bridge Interface
	   $ sudo ip link set veth-br0-green master br0

# Up the bridge end of the newly created virtual cable (for Green namespace). 
	   $ sudo ip link set veth-br0-green up

# Assign IP address in the Red namespace. Here ip netns exec red means, the command added afterward, will be executed from inside Red namespace.
	$ sudo ip netns exec red ip addr add 172.16.10.1/24 dev veth-red

# UP the interface of Red namespace.
	$ sudo ip netns exec red ip link set veth-red up

# Assign IP address in the Green namespace. Here ip netns exec Green means, the command added afterward, will be executed from inside Green namespace
	$ sudo ip netns exec green ip addr add 172.16.10.2/24 dev veth-green

# UP the interface of Green namespace
	$ sudo ip netns exec green ip link set veth-green up

# Add default Route to red namespace. This means, if no other routing rule is present, system will use this route by default. In other word, the bridge interface(172.16.10.10) is declared as gateway for Red namespace.
   
	$  sudo ip netns exec red ip route add default via 172.16.10.10


# Add default Route to Green namespace. This means, if no other routing rule is present system will use this route by default. In other word, bridge interface (172.16.10.10) is declared as gateway for Green namespace.

	$ sudo ip netns exec green ip route add default via 172.16.10.10

# Successfully Ping Green namespace (IP: 172.16.10.2) from Red namespace.
	$ sudo ip netns exec red ping 172.16.10.2

# Successfully Ping Red namespace from Green namespace.
	$ sudo ip netns exec green ping 172.16.10.1

# Now an iptable rule is needed to add as both namespace needs to communicate to outside. The rule will be included in NAT table (as, local IP will communicate to outside world) and in POSTROUTING chain (i.e. POSTROUTING chain is typically used for outgoing packets after routing decission has been made) and afterward, MASQUERADE is used for dynamic source NAT which allows the system to automatically modify the source IP address of the outgoing packets to match the address of the outgoing interface.

	$ sudo iptables -t nat -A POSTROUTING -s 172.16.10.0/24 -j MASQUERADE

# Successfully Ping to the Outside world (i.e. google DNS) from Red namespace
	$ sudo ip netns exec red ping 8.8.8.8

# Successfully Ping to the Outside world (i.e. google DNS) from Green namespace
	$ sudo ip netns exec Green ping 8.8.8.8

