## ERLUSOO
### Enforced Remote Learning Using Squid On OpenWRT

#### Description

Your kid is stuck at home and is supposed to be reading and doing school work with whatever minimal guidance your teachers are able to provide. But most of the day they watch Youtube, play games you've never heard of (and some you have), and generally waste their day online. Wouldn't it be nice if they were only able to vists sites specifically needed for their school work?

Using the principle of least priviledge and a few weeks of feedback from my kids, I've constructed this squid proxy config which runs on most [openWRT](https://openwrt.org) routers. Since it runs on your router and not your child's device, squid can block traffic from any operating system: GNU/Linux, Windows, macOS, ChromeOS (Chromebook), Android, iOS, etc. Maybe you've found it hard to configure parental controls when your kid has so many different devices? Maybe you would like to consolidate rules for all kids and all devices in one place? ERLUSOO is for you.

#### Pre-requisites

1. No other wifi/Internet options available to the child (such as a phone with mobile data, phone hotspot, a neighbor's wifi, etc.)
1. Home router running opernWRT 19.07 (maybe older versions work - you tell me)
1. External drive to store squid's cache files
1. You made a self-signed CA [certificate](https://wiki.squid-cache.org/Features/DynamicSslCert)

#### Assumes
1. The `whitelist.txt` file is currently tailored for Google Classroom and other random educational sites. You can easily add your own and remove sites that your kids don't need.
1. You have two children - only needed in `/etc/config/firewall`. You may add or remove kiddos from this file as needed. This allows you to selectively turn off blocking for one or both children, though then they'll be able to watch Youtube again because of the dastardly client-side caching they do. Don't even get me started on Google Doodles!#@##!!%
1. Your router's IP address is 192.168.1.1. You can change this with a search and replace in `/etc/config/firewall`.
1. You have a guest wifi network - only needed in `/etc/config/firewall`. You may remove the guest redirect rules if you don't have one.
1. You know your child's device MAC address. This is usually available in the network settings on their device.
1. You will mount a hard disk to `/tmp/squid`.
1. You put your CA private key and certificate into one file called `/etc/squid/certs/myCA.pem`.
1. You don't mind constantly adding sites to `/etc/squid/whitelist.txt` as teachers hand out assignments that require new random websites. Good luck blocking Youtube. Don't forget to restart squid.
1. You already have `ssh` access to your router and are comfortable with the Linux command line.

### Instructions
#### On your openWRT router
1. Install squid using these [instructions](https://openwrt.org/docs/guide-user/services/proxy/proxy.squid) (don't bother with the ```Set up forwarding``` secion, we'll do that in Step 5) and replace `/etc/squid/squid.conf` with this version
1. Copy `whitelist.txt` to `/etc/squid/whitelist.txt`
1. Copy `myCA.pem` to `/etc/squid/certs/`
1. Create DH parameters with `openssl dhparam -outform PEM -out /etc/squid/certs/dhparam.pem 2048`
1. Append the config in `/etc/config/firewall` to the version on your router. Find and replace xx:xx:xx:xx:xx:xx and yy:yy:yy:yy:yy:yy with actual MAC addresses, remove/add devices or kids, change router IP address from `192.168.1.1`, go crazy with it
1. restart squid with `service squid restart` and the firewall with `service firewall restart`

#### On your child's device(s)
1. From your kid's device, you should see certificate errors in the browser if you load a whitelisted site. This means that the firewall on your router is sending traffic from your kid's device to squid, and squid is now in control. Your kid's browser just doesn't trust the squid certificate yet and that's why it's freaking out. Sites that aren't on the whitelist should show an Access Denied page. This is a small victory. Savor it. You're almost to the finish line. 
1. Instal the self-signed certificate you generated (`myCA.der`) into the browser of your child's device as a Certificate Authority (this process varies depending on which operating system your child's device uses.)
1. Clear all browsing history and data so that you can make sure your kid's most important sites work. You might need the kid for this next step.
1. Visit a site on the whitelist; you should not see any errors, the page should load normally with the exception of whatever ads might be present on the page. They are blocked as a nice side effect. 
1. If the page partially loads or doesn't work, check out the step below to watch for blocked sites and add them one by one until the page works fully. Don't forget to restart squid.
1. As your child tries to work they will encounter legitimate sites that are blocked. Tell them to come to you right away instead of staring off into space all day or reading *The Lightning Thief* for the 23rd time. Use this to find out which ones are blocked and add them to `/etc/squid/whitelist.txt`:
```bash
tail -F /tmp/squid/log/access.log \
|grep 403 \
|grep -v play.google.com \
|grep -v IP_ADDRESS_OF_OTHER_CHILD(REN)_IF_YOU_HAVE_MORE_THAN_ONE_CHILD
```

Don't forget to restart squid.

And put some fun messages in /usr/share/squid/errors/templates/ERR_ACCESS_DENIED because they will see this page a lot! Really spice things up to show them you care about their education.

### Caveats

1. As mentioned, if they get Internet access through other means it's game over and you're back to square one.
1. If they convince you to disable the blockade (which you can do quickly via the firewall port redirect rules) so they can quickly check on just one thing don't fall for it. As soon as you turn your back they are going to Youtube, the browser will cache the application, and it will connect directly to IP addresses which are not blocked by this squid config. Ask me how I know. Wonder what I'm working on next?
1. Lots of other cacheing can render this ineffective (kids can and have downloaded multiple seasons worth of shows on Netflix before you set this up. Ask me how I know.) You may want to do a sweep of their favorite apps after this is setup. Chromebooks have a great feature called [Powerwash](https://support.google.com/chromebook/answer/183084?hl=en) that makes this very simple.

### Contributing

Pull requests are welcome if you think of something I didn't. In particular this Readme probably glosses over some detail or another. If I left something out please let me know. I probably won't accept PRs for `whitelist.txt` since it is highly dependent on what my kids need day-to-day and it works for us as is.