---
title: "How to Proxy Xamarin Mobile Apps"
date: "2022-10-28"
tags: ["redteam"]
---

> This blog post assumes you’re working with physical rooted devices, digital devices may work but not tested.
> 

If you’ve spent a bit of time testing mobile applications, chances are you’ve come across a mobile application that was built using Xamarin. Xamarin is a cross-platform application building tool for building iOS and Android mobile applications. This platform is becoming more and more common as an app building tool.

### What’s the problem?

All apps built in Xamarin by default ignore proxy settings that are set on the mobile device. A critical part of performing any testing on mobile applications is viewing and modifying requests before they reach the server. In non-Xamarin mobile applications, this is completed by using the proxy settings on the device and capturing the packets with your favourite tool (eg Burp proxy).

### Android Bypass

If you’re just testing the Android side of the application, there’s a very simple bypass that gets around these proxy settings with a few commands.

Note: Burp is set up on a VM with a bridged IP address (192.168.0.210 in this case) and configured to receive connections on this IP instead of 127.0.0.1.

```jsx
# Connect to device
adb.exe shell

# Switch to root
su

# Force all communication on port 80 to local kali instance running burp on 8080 (change IP to your kali IP)
iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination 192.168.0.210:8080

# Force all communication on port 443 to local kali instance running burp on 8080 (change IP to your kali IP)
iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination 192.168.0.210:8080
```

Assuming you’ve got your burp certificate set up, you should now see the traffic in burpsuite.

### iOS Proxy

Setting up a proxy for Xamarin apps on iOS is a bit more work unfortunately. There are many ways to do this, I’ve found this to be the easiest with the least amount of headaches. You need to set up a VPN server on a virtual machine (could be the same Kali box you’re using to run burp - in my case it isn’t due to my Kali box giving me errors at the time) and then connect the iOS device to that VPN. This forces all traffic to go through your VPN which you can then funnel into burp. 

```jsx
# Pull the latest install script from https://github.com/angristan/openvpn-install
curl -O https://raw.githubusercontent.com/angristan/openvpn-install/master/openvpn-install.sh

# Allow it to be executable 
chmod +x openvpn-install.sh

# Run the script
./openvpn-install.sh

# Proceed through the steps making sure you set up the IP as your internal IP (mine was 192.168.0.123) and pretty much select the defaults

# Confirm Openvpn service is running
sudo service openvpn status
```

With the VPN set up on the host you should now have a .ovpn file you can use on your iOS device. I chose to transfer this with SCP as SSH was set up on my device already.

```jsx
scp kalivpn.ovpn root@192.168.0.200:/path/on/device
```

Install OpenVPN on the iOS device, I chose to copy over an .apk file over and install it using Filza instead of dealing with the App Store. Then, install the profile you received when you set up the OpenVPN server on your VM.

Now, we’ve got our iOS device sending traffic to our VM that is running OpenVPN. Great! Let’s get it to now go into burp.

To do this we need to run a few iptables commands on the linux box that is running the OpenVPN server to force the traffic from the OpenVPN box to Burpsuite. 

```jsx
# Sending all traffic from port 80 through to my burp proxy on 192.168.0.210 (this should be 127.0.0.1 if you're running the VPN on the same host as Burp)
sudo iptables -t nat -A PREROUTING -i tun0 -p tcp — dport 80 -j DNAT — to-destination 192.168.0.210:8080

# Sending all traffic from port 443 through to my burp proxy on 192.168.0.210 (this should be 127.0.0.1 if you're running the VPN on the same host as Burp)
sudo iptables -t nat -A PREROUTING -i tun0 -p tcp — dport 443 -j DNAT — to-destination 192.168.0.210:8080
```

Finally, you just need to enable invisible proxying in burp.

Within Burp go to ```Proxy > Options > Select Proxy Listener > Edit > Request Handling > Support Invisible Proxying``` Assuming you have the burp certificate installed and there’s no certificate pinning enabled on this application you should see the traffic flowing through burp.

This technique can also be used for Android in the same way, however I personally opt for the easier way.

### Conclusion

Xamarin apps appear to be more and more common these days, however, once you set this up once you should be good to go for any future apps after that. I think these methods are a lot nicer than using mitmproxy and a Digital Ocean box.

Thanks for reading. 

Peace.

Credit Salman Syed’s Medium post that helped me grasp the iOS side of things originally.

[https://slmnsd552.medium.com/how-to-capture-non-proxy-aware-mobile-application-traffic-ios-android-xamarin-flutter-924fe044facf](https://slmnsd552.medium.com/how-to-capture-non-proxy-aware-mobile-application-traffic-ios-android-xamarin-flutter-924fe044facf)
