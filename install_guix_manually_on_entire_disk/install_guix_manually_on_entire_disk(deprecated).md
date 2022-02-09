# install Guix manually on entire disk(deprecated)

I just install GUN guix on my ASUS entire disk today. This is lack of documents about it, so I'm recording it by this article.

## reference
1. https://guix.gnu.org/manual/en/html_node/Manual-Installation.html#Manual-Installation

## step

### Networking(wireless connection)
run the following command to find the network interface about wireless named begin with wl, like `wlp3s0`.
```
# ifconfig -a      # or "ip address"
```

#### wireless connection
To configure wireless networking, you can create a configuration file for the `wpa_supplicant` configuration tool (its location is not important) using one of the available text editors such as `nano`:
```
nano wpa_supplicant.conf
```

Here is sample of `wpa_supplicant.conf`:
<pre>
  network={
    ssid="<em>my-ssid</em>"
    key_mgmt=WPA-PSK
    psk="<em>the network's secret passphrase</em>"
  }
</pre>

Start the wireless service and run it in the background with the following command (substitute interface with the name of the network *interface* like `wlp3s0` you want to use):
```
wpa_supplicant -c wpa_supplicant.conf -i interface -B
```

At this point, you need to acquire an IP address. On a network where IP addresses are automatically assigned *via* DHCP, you can run:
<pre>
  dhclient -v <em>interface</em>
</pre>