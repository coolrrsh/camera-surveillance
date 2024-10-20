
connect pins RX, TX, GND of serial UART to Raspberry UART pins.

flash buster headless image on sd card using

```
sudo sh -c "xzcat image.wic.xz | dd of=/dev/sdx bs=8M"
```

Once flashing is completed, re-mount the SD card to PC and execute the following command.

``` 
echo >> "enable_uart=1" /etc/bootfs/config.txt

echo "pi:$(echo 'raspberry' | openssl passwd -6 -stdin)" > userconf.txt
```

Append the following console args in boot/cmdline.txt

```
console=serial0,115200

```

Insert the SD card in Raspberry Pi board.

```
user: pi
password: raspberry

```

Sharing PC internet to Raspberry Pi via Ethernet ->

1. Enable IP Forwarding sysctl -w net.ipv4.ip_forward=1. This will enable the kernel to forward packets, which are arriving to this machine.

2. Assign a static IP to the ethernet interface, in my case eth1. sudo ifconfig eth1 192.168.122.10 netmask 255.255.255.0 up Just make sure that no other device on your network uses this ip. On the safer side, assign the ip in a totally different subnet.

3. Enable masquerading on the interface which is connected to the internet. In my case, its my WiFi interface, eth0. sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE. This will masquerade (replace the src ip on the packet with the eth0 ip) all traffic arriving from other interfaces, to the eth0 interface.

4. Add iptable rules to ACCEPT and FORWARD traffic from the subnet

```
sudo iptables -I FORWARD -o eth0 -s 192.168.0.0/16 -j ACCEPT
sudo iptables -I INPUT -s 192.168.0.0/16 -j ACCEPT

```


These rules will redirect the kernel to accept packets coming from the 192.168.0.0 subnet, and forward them onto the eth0 interface.
(wlo1 instead of eth0 if wifi is on wlo1)

Open the ethernet wired setting in Ubuntu, select ipv4 option > manual 

```
ip: 192.168.218.10 netmask:255.255.255.0 gateway:<ip of wlo1> 
dns: 8.8.8.8, 8.8.4.4
```

`CAPTURE VIDEO`

```
libcamera-vid --width 1920 --height 1080 -o full_hd.h264
```
