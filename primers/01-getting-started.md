# Overview

This is a walkthrough for how to get started with receiving multicast traffic in a web app.

# Implementations

In 2021-06 there's only 1 implementtion yet, which is a demo fork of chromium that does not yet have the security model implementation.

## How to Install

The latest binary .deb installer (from a patch applied to a recent chromium stable build) can be downloaded from the top link here:

 * [CURRENT_BINARIES.stable.md](https://github.com/GrumpyOldTroll/chromium_fork/blob/main/CURRENT_BINARIES.stable.md)

~~~
URL=$(curl https://raw.githubusercontent.com/GrumpyOldTroll/chromium_fork/main/CURRENT_BINARIES.stable.md | grep https | head -n 1 | cut -f3 -d " ")
curl -O ${URL}
sudo apt install ./$(basename ${URL})
~~~

To remove it later:

~~~
sudo apt remove chromium-browser-mc-unstable
~~~

## How to Build (optional)

If you want to build the installer yourself instead of downloading the binary.  You need plenty of RAM and disk space (at least ~8 and ~100 GB respectively in 2021-06, but it may grow).  It takes something like 10 hours.

### Ubuntu/Debian

(tested on Ubuntu 18.04 and 20.04)

~~~
sudo apt-get install -y docker.io screen snapd git python3 python-is-python3
sudo snap install jq
git clone https://github.com/GrumpyOldTroll/chromium_fork/
sudo usermod -aG docker ubuntu
# log out and back in
CHAN=stable
VER=$(curl -s -q -f https://raw.githubusercontent.com/GrumpyOldTroll/chromium_fork/main/LAST_GOOD.${CHAN}.sh | grep LAST_GOOD_TAG | cut -f2 -d=)
cd chromium_fork
CHAN=${CHAN} VER=${VER} ./nightly.sh
~~~

### Cloud build (AWS)

Also possible is building on a cloud device.  (NB: This can't be done on free tier, the machines don't have enough ram, and the disk space also needs to be bigger.)

These will all try to pick decent defaults, but if you don't have a key pair in the default region (us-west-1) or the first key-pair isn't the one you've got loaded with ssh-add it won't work if nothing is set explicitly, so you may need to set one or more variables:

Most likely to be necessary:

 * [REGION](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html#Concepts.RegionsAndAvailabilityZones.Regions) (AWS region)
 * [KEYPAIR](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) (keypair for connecting to the instance)

If you have a preference or you're aiming for something specific, several others are possible, for instance:

 * [INSTTYPE](https://aws.amazon.com/ec2/instance-types/) (the instance type)
 * [AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html) (image id).  NB: the same release in different regions is a different id
 * CHAN: stable or dev
 * VER: version (recommended: LAST_GOOD_TAG from the [fork](https://github.com/GrumpyOldTroll/chromium_fork/blob/main/LAST_GOOD.stable.sh))

Also setting 'KEEP' to anything besides 0 will keep the instance alive afterwards (this will have [extra costs](https://aws.amazon.com/ec2/pricing/on-demand/) and you'll need to manually clean it up).

Note also that manual cleanup may be needed if anything goes wrong.  This should make a single instance of INSTTYPE (default=c5.2xlarge) in REGION (default=us-west-1) and a single security group named `cbuild-sg-<timestamp>.

~~~
curl -O https://github.com/GrumpyOldTroll/chromium_fork/blob/main/aws-build.sh
chmod +x aws-build.sh
KEYPAIR=my-key REGION=us-west-2 ./aws-build.sh
~~~

# Traffic Sources

In order to receive multicast, you'll need to have a source of multicast traffic sending from somewhere you can receive from.

## Running a Local Sender

There's several ways to send traffic locally, and a few possibilities for where to run it from.

### External Machine on Same LAN

If you've got your receiving device physically connected to a LAN and an external device on the receiver's default route, that's one of the easiest and most normal realistic cases.  This one doesn't need any special routes, but it does need a separate device.  Some VM setups can do this between VMs, but not always.

From the sender:

~~~
curl -O https://raw.githubusercontent.com/GrumpyOldTroll/libmcrx/master/test/send.py
chmod +x send.py
./send.py -g 232.2.2.2 -p 6000 -i 500 -d 7200 &
./send.py -g ff3e::8002:202 -p 6000 -i 500 -d 7200 &
~~~

A tcpdump should show these packets, either on the sender or the receiver:

~~~
sudo tcpdump -n udp and ip dst 232.2.2.2
sudo tcpdump -n udp and ip6 dst ff3e::8002:202
~~~

If the sender has multiple interfaces and you want the sending on the non-default interface, you might have to add a route on the sender, giving it the output interface's device name for SSM addresses.  You can check which interface is the output, either with tcpdump or by checking the route:

~~~
ip route get 232.2.2.2
ip -6 route get ff3e::8002:202
~~~

If you need to change the output interface, setting the route for SSM addresses should do it (here the interface is named `xdn0`):

~~~
sudo ip -6 route add ff3e::/96 dev xdn0
sudo ip route add 232.0.0.0/8 dev xdn0
~~~

If the packets are being sent but the receiver can't see them, it might mean you need to do a join first, or it might mean you'll need to use another method.

At this point receiving should work with the steps in the "[Receiving Traffic](#receiving-traffic)" section, e.g. on the [demo page](https://grumpyoldtroll.github.io/wicg-multicast-receiver-api/demo-multicast-receive-api.html) or with `mcrx-check`:

~~~
SRC=10.7.1.1
./mcrx-check -s ${SRC} -g 232.2.2.2 -p 6000 -c 50 -d 0 -v
~~~

### Internal Sender on Same Machine

This doesn't require any external machines or traffic, which can be helpful for testing and development, but it does require setting local routes to get the traffic going the right way.

The route for the destination group address needs to go to the sending interface, and the route for the source interface needs to go thru the receiving interface.

There are a few different options, but perhaps the easiest is to run the sener in a [netns](https://man7.org/linux/man-pages/man8/ip-netns.8.html) connected by a [veth pair](https://man7.org/linux/man-pages/man4/veth.4.html).  It can also be ok to just set the route for the send traffic on a veth pair without using a netns. (NB: For IPv4 I've also seen loopback interfaces work, but IPv6 had trouble with sending and receiving on the same interface.)

~~~
sudo << EOF
ip netns add sender
ip link add vtx0 type veth peer name vrx0
ip addr add 10.20.20.2/24 dev vrx0 
ip link set dev vtx0 netns sender
ip netns exec sender ip addr add 10.20.20.1/24 dev vtx0
ip netns exec sender ip link set up dev vtx0
ip netns exec sender ip route add default dev vtx0
EOF
sudo ip netns exec sender python3 src2/libmcrx/test/send.py -g 232.2.2.2 -p 6000 -i 500 -d 7200 &
~~~

At this point receiving should work with the steps in the "[Receiving Traffic](#receiving-traffic)" section, e.g. on the [demo page](https://grumpyoldtroll.github.io/wicg-multicast-receiver-api/demo-multicast-receive-api.html) or with `mcrx-check`:

~~~
SRC=10.20.20.1
./mcrx-check -s ${SRC} -g 232.2.2.2 -p 6000 -c 50 -d 0 -v
~~~

### Other Senders

In addition or instead of running `send.py` or similar test traffic, it's also possible to run video traffic that can be consumed by [VLC](https://www.videolan.org/) and probably [ffmpeg](https://www.ffmpeg.org/).

~~~
vlc -vvv ./vid.mp4 --sout '#udp{dst=232.3.3.3,port=5004,mux=ts}' --ttl 15 --loop --file-caching 10000
~~~

~~~
ffmpeg -re -i vid.mp4 -c copy -f mpegts udp://232.10.10.2:12000?pkt_size=1316
~~~

## Tunneling a Remote Sender

These instructions are to sort of manually set up the network to tunnel the external multicast traffic into the local machine.

It's possible to set this all up externally.  Ideally your network will be the one that does this, in which case you have nothing to do here, your browser can just receive the traffic.

But until your network gets to that, you can still tunnel the traffic in yourself.

For advanced and persistent setup, you can stand up a multicast-capable network that does the ingest for you.  There are instructions in the [multicast-ingest-platform](https://github.com/GrumpyOldTroll/multicast-ingest-platform/blob/master/sample-network/README.md) repo.  The sections below describe how to do the same thing manually for your local machine.

### Internet2 Sources

Internet2 has several public relays running.  To connect to one of the relays, run an [AMT](https://www.rfc-editor.org/rfc/rfc7450.html) gateway instance:

~~~
AMTIP=$(dig +short amt-relay.m2icast.net | head -n 1)
sudo docker run -d --rm --name amtgw --privileged grumpyoldtroll/amtgw ${AMTIP}
~~~

There's a bunch of sources sending traffic reachable via those relays.  There's a [catalogue](https://multicastmenu.herokuapp.com/) that's kept more or less up to date.  Many of the entries say their UDP port, and most of them use `1234`.

You can pick a source and group from that catalogue:

~~~
SRC=162.250.138.201
GRP=232.162.250.141
PORT=1234
~~~

You also need the source to be routed via the gateway so the join will go in that direction:

~~~
sudo ip route add ${SRC}/32 dev docker0
~~~

At this point receiving should work with the steps in the "[Receiving Traffic](#receiving-traffic)" section, e.g. on the [demo page](https://grumpyoldtroll.github.io/wicg-multicast-receiver-api/demo-multicast-receive-api.html) or with `mcrx-check`:

~~~
./mcrx-check -s ${SRC} -g ${GRP} -p ${PORT} -c 50 -d 0 -v
~~~

### DRIAD Sources

There are several sources that are publicly reachable whose relays are discoverable with [DRIAD](https://www.rfc-editor.org/rfc/rfc8777.html).

You can pull in traffic from those by discovering the relay and setting the route for the source of the (S,G) to go through the docker interface:

~~~
SRC=23.212.185.14
GRP=232.1.1.1
PORT=5001
~~~

Setting up the docker container needs to do a DRIAD lookup:

~~~
curl -O https://raw.githubusercontent.com/GrumpyOldTroll/libmcrx/master/driad.py
AMTIP=$(python3 driad ${SRCIP})
sudo docker run -d --rm --name amtgw --privileged grumpyoldtroll/amtgw ${AMTIP}
~~~

And you need the source IP routed to the docker container:

~~~
sudo ip route add ${SRC}/32 dev docker0
~~~

At this point receiving should work with the steps in the "[Receiving Traffic](#receiving-traffic)" section, e.g. on the [demo page](https://grumpyoldtroll.github.io/wicg-multicast-receiver-api/demo-multicast-receive-api.html) or with `mcrx-check`:

~~~
./mcrx-check -s ${SRC} -g ${GRP} -p ${PORT} -c 50 -d 0 -v
~~~

# Receiving Traffic

~~~
URL="https://grumpyoldtroll.github.io/wicg-multicast-receiver-api/demo-multicast-receive-api.html"
chromium-browser-mc-unstable --enable-blink-features=MulticastTransport ${URL}
~~~

## Network Troubleshooting with Simpler Receiver

If it's all working right, the browser receive will work.  But to check whether it's the networking or something in the browser build, it's also possible to use the underling receive library, [libmcrx](https://github.com/GrumpyOldTroll/libmcrx) for a more isolated test.  There are many ways the browser test can go wrong that this one cannot, but if this one fails the browser test will not succeed.

In either case, you'll need the source IP address of those packets, and you pass it to the libmcrx testing program like this:

~~~
git clone https://github.com/GrumpyOldTroll/libmcrx
cd libmcrx
./autogen.sh
./configure
make
~~~

Running is like this:

~~~
SRC=10.7.1.1
./mcrx-check -s ${SRC} -g 232.2.2.2 -p 6000 -c 50 -d 0 -v
~~~

The output should look like this:

~~~
06-27 16:25:48: joined to 10.7.1.1->232.2.2.2:6000 for 2s, 2 pkts received
06-27 16:25:50: joined to 10.7.1.1->232.2.2.2:6000 for 4s, 6 pkts received
06-27 16:25:52: joined to 10.7.1.1->232.2.2.2:6000 for 6s, 10 pkts received
~~~

If that's not working, the first thing to check is whether the join is going out on the right interface.  From another shell, tcpdump for igmp (for v4) or mld (for v6) on the receiving device:

~~~
sudo tcpdump -n -vv igmp
~~~

Output looks like this when joining if it's working:

~~~
sudo tcpdump -n -vv igmp
tcpdump: listening on enx00e04cab9b3b, link-type EN10MB (Ethernet), capture size 262144 bytes
16:47:24.578717 IP (tos 0xc0, ttl 1, id 0, offset 0, flags [DF], proto IGMP (2), length 44, options (RA))
    10.7.1.58 > 224.0.0.22: igmp v3 report, 1 group record(s) [gaddr 232.2.2.2 allow { 10.7.1.1 }]
16:47:24.894738 IP (tos 0xc0, ttl 1, id 0, offset 0, flags [DF], proto IGMP (2), length 44, options (RA))
    10.7.1.58 > 224.0.0.22: igmp v3 report, 1 group record(s) [gaddr 232.2.2.2 allow { 10.7.1.1 }]
~~~

And like this when leaving (by e.g. killing the receiver):

~~~
16:47:32.186650 IP (tos 0xc0, ttl 1, id 0, offset 0, flags [DF], proto IGMP (2), length 44, options (RA))
    10.7.1.58 > 224.0.0.22: igmp v3 report, 1 group record(s) [gaddr 232.2.2.2 block { 10.7.1.1 }]
16:47:32.188429 IP (tos 0xc0, ttl 1, id 19185, offset 0, flags [DF], proto IGMP (2), length 40, options (RA))
    10.7.1.1 > 232.2.2.2: igmp query v3 [max resp time 1.0s] [gaddr 232.2.2.2 { 10.7.1.1 }]
16:47:32.862698 IP (tos 0xc0, ttl 1, id 0, offset 0, flags [DF], proto IGMP (2), length 44, options (RA))
    10.7.1.58 > 224.0.0.22: igmp v3 report, 1 group record(s) [gaddr 232.2.2.2 block { 10.7.1.1 }]
16:47:32.864340 IP (tos 0xc0, ttl 1, id 19351, offset 0, flags [DF], proto IGMP (2), length 40, options (RA))
    10.7.1.1 > 232.2.2.2: igmp query v3 [max resp time 1.0s] [gaddr 232.2.2.2 { 10.7.1.1 }]
~~~

Or for IPv6, the tcpdump filter implementation as of 2021-06 is less mature:

~~~
sudo tcpdump -n -vv icmp6 and ip[40]==128
~~~

(NB: seeing some trouble in ubuntu 20.04, MLD apparently not going out.)

If you're seeing packets in tcpdump but not in the packet count, the first thing to check is whether the destination UDP port matches what the receiver is listening to.  After that it gets trickier.

