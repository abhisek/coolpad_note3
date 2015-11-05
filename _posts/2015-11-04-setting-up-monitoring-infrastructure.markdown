---
layout: post
title:  "Setting Up Monitoring Infrastructure"
date:   2015-11-04 20:14:00 +0530
categories: analysis
---
The objective for the monitoring infrastructure was to visualize network traffic between the phone
and the Internet. The setup should also be able to redirect HTTP and HTTPS traffic to other tools
for better analysis. I could manage to setup a monitoring infrastructure for the phone using the
following hardware:

* 1 Linux laptop with 2 network interface:
  * Ethernet
  * WiFi
* 2 WiFi Router
  * Home WiFi Router
  * Additional WiFi router for connecting phone
* Coolpad Note 3 (phone)

The monitoring infrastructure looks conceptually like this:

![Monitoring Infrastructure]({{ site.baseurl }}/images/mon1.jpg)

The Linux laptop is connected to the Home WiFi router for Internet access without any special configuration. I used an additional WiFi router (2nd WiFi router) to connect the phone while the Linux laptop acting as the NAT gateway. The 2nd WiFi router's upstream port is connected with Linux laptop's ethernet port. The phone is configured to use *only* the 2nd WiFi router's SSID. The setup ensures that any traffic from the phone *must* pass through the Linux laptop.

For NAT to work, following *iptables* rule is required:

{% highlight bash %}
#!/bin/bash

IFACE_IN="eth1"
IFACE_OUT="wlan0"

# Setup Eth (IFACE_IN) using network manager
#ifconfig $IFACE_IN up
#ifconfig $IFACE_IN 10.111.0.1 netmask 255.255.255.0

# Connect to WiFi (IFACE_OUT) so that wlan0 is connect to home LAN

iptables -F
iptables -t nat -F

# NAT
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o $IFACE_OUT -j MASQUERADE
iptables -A FORWARD -i $IFACE_IN -o $IFACE_OUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i $IFACE_IN -o $IFACE_OUT -j ACCEPT
{% endhighlight %}

This setup can be verified by running *Wireshark* in the Linux laptop and observing data originating from the phone. The **Coolpad Note 3** makes multiple HTTP connection to vendor sites so it should not be difficult to verify monitoring of phone traffic passing through the laptop.

However, in order to inspect HTTPS traffic, we need to redirect the HTTPS connection to our interceptor that can perform SSL MiTM. In my setup, I used Burp as the SSL interception proxy. Following additional *iptables* rules are required in order to redirect HTTP and HTTPS ports.

{% highlight bash %}
# Transparent proxy for 80,443  (IP of external interface required here)
iptables -t nat -A PREROUTING -i $IFACE_IN -p tcp -m multiport --dports 80,443 -j DNAT --to-destination 10.111.0.1:8080
iptables -t nat -A PREROUTING -i $IFACE_OUT -p tcp -m multiport --dports 80,443 -j REDIRECT --to-port 8080
{% endhighlight %}

**The above rules assume Burp is running in the Laptop and listening on port 8080**

However, for Burp's SSL interception to work, Burp's CA certificate must be installed in the device. Follow PortSwigger guide on [Installing Burp CA Certificate in Android Device](https://support.portswigger.net/customer/portal/articles/1841102-installing-burp-s-ca-certificate-in-an-android-device). 

> Note: SSL interception will not work even after installing Burp CA certificate for apps that uses [SSL Pinning](https://www.owasp.org/index.php/Certificate_and_Public_Key_Pinning)

I was able to confirm Burp intercepting both HTTP and HTTPS traffic after installing Burp's CA certificate in the device and setting up appropriate firewall rules for NAT and HTTP/HTTPS redirection to Burp listener.

![Burp Interception]({{ site.baseurl }}/images/burp1.png)

It is important to keep Wireshark running in the Laptop to identify HTTP or HTTPS traffic to non-standard ports. For such cases, appropriate redirection rule for non-standard ports has to be added.
