# Tor Script

After a Cyber Security Awareness Training for company X, I thought about sharing the idea and even the script used to make it easier and available for everyone.

![header](static/header.png)

### Before doign anything, you can check here some Tor uses/users statistics 05/2021 


* By Users : 
![stats](https://metrics.torproject.org/userstats-relay-country.png?start=2021-03-03&end=2021-06-01&country=all&events=off)

* By Country : 
![stats](https://i.imgur.com/suySqtJ.png)

* By Relays : 
![stats](https://metrics.torproject.org/networksize.png?start=2021-03-03&end=2021-06-01)

* By Relays : 
![stats](https://metrics.torproject.org/bandwidth-flags.png?start=2021-03-03&end=2021-06-01)


## Starting with docker 

So, here we'll use a docker image with Tor installed on it. We

# Docker 

On docker I'm going to use alpine instead of Debian on docker for it's light weight.

## Configuring the image

starting with tor config file `torrc` / (`/etc/tor/torrc`)
```
    VirtualAddrNetwork 0.0.0.0/10
    AutomapHostsOnResolve 1
    DNSPort 0.0.0.0:53530
    SocksPort 0.0.0.0:9050
```
> you can change port 1962 to your own

![Config](https://i.imgur.com/wewtou6.png)

and now the `Dockerfile`

```
FROM alpine:latest
RUN apk update && apk add tor
COPY torrc /etc/tor/torrc
RUN chown -R tor /etc/tor
USER tor
ENTRYPOINT ["tor"]
CMD ["-f", "/etc/tor/torrc"]
```
![Dockerfile](https://i.imgur.com/PbplMVn.png)

* The containing of the folder should be :

![output](https://i.imgur.com/wFmH7Sv.png)


Now let's build and image : `docker build -t sofiane/tor .`

![Built](https://i.imgur.com/LNLGq6c.png)

Check the image `docker image ls | grep sofiane/tor

![check](https://i.imgur.com/FAmPRFo.png)

## Using the proxy

Start by running the docker image `docker run --rm --detach --name tor --publish 1962:1962 sofiane/tor`

![](https://i.imgur.com/Ub5Rljr.png)

Now let's test it out!

* Without Proxy : My Real IP 
![noproxy](https://i.imgur.com/MHLZKzv.png)
* With Proxy : a Tor exit
![proxy](https://i.imgur.com/po3SHo2.png)

You can check with tor website too : 
`curl --socks5 localhost:9050 --socks5-hostname localhost:9050 -s https://check.torproject.org/ | cat | grep -m 1 Congratulations | xargs`

![](https://i.imgur.com/NQ3TeW9.png)


## Configuring the VPN

We won't use the VPN on a docker, because first we need to create the `tun` device on the container which is a kill for the Security.

So to setup as a vpn, we'll use a Linux VPS ( Debian )

For the VPN, you will always use the same Tor config file!

But, you'll need to make some changes to the iptables rules.

> these rules are for the transparently, what we call `Transparent Routing Traffic Through Tor`

> Check Tor website explaining this in details : [TransparentProxy
](https://gitlab.torproject.org/legacy/trac/-/wikis/doc/TransparentProxy#WARNING)


First of all, add these 3 Environment variables
![](https://i.imgur.com/x9p07gZ.png)
And the iptables rules :

![](https://i.imgur.com/VYDJ15N.png)

don't forget that you need `openvpn`, `iptables` and `tor` installed on your machine.

Final step, is to create your own openvpn profile, to do that I suggest you this small script that I love and use often : 

![](https://i.imgur.com/Wsqahwg.png)

```
$ curl -O https://raw.githubusercontent.com/angristan/openvpn-install/master/openvpn-install.sh
$ chmod +x openvpn-install.sh
```

and run it using 
```
$ ./openvpn-install.sh
```

and for setting the rules, we will use this script :

![](https://i.imgur.com/MvnbLzC.png)


Okey, now let's do this together ! 

- 1 - connect to the vps ( don't forget to allow traffic on the used ports)
- 2 - install all the needed packages
![](https://i.imgur.com/wdzg1TG.png)
- 3 - change the `torrc` file
    ```
    curl -L https://raw.githubusercontent.com/SofianeHamlaoui/Tor-scripts/main/torrc > /etc/tor/torrc > torrc && sudo mv torrc /etc/tor/torrc
    ```
    ![](https://i.imgur.com/6hMsYXP.png)

- 4 - Using the openvpn script 

    ![](https://i.imgur.com/ZZqmuKr.png)

    and save the `.ovpn` file

- 5 - Enabling OpenVpn & Tor services :
![](https://i.imgur.com/P4b479p.png)

- 6 - Adding the rules
    ```
    $ curl -O https://raw.githubusercontent.com/SofianeHamlaoui/Tor-scripts/main/vpn.sh && chmod +x vpn.sh
    $ sudo ./vpn.sh
    ```