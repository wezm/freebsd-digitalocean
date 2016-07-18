# freebsd-digitalocean
Lightweight, zero-dependency, self-configuration for FreeBSD droplets on DigitalOcean

## Features
- Allows you to run a FreeBSD droplet that auto-configures itself at boot time.
- No dependency on extra packages or ports -- it's just a simple shell script.
- Replaces all your /etc/rc.conf networking configurations with just one `digitalocean="YES"` setting.
- Supports IPv4 and IPv6 public and private network addressing and routing.
- Supports Floating IPs by automatically adding the corresponding private anchor IPs for you.
- Automatically sets the hostname.
- Automatically sets the *freeebsd* user's authorized public keys.
- Automatically adds the droplet's DNS nameservers.
- Completely configurable.

## Installation
As root:
- Copy digitalocean to /usr/local/etc/rc.d (chmod 700)
- Copy digitalocean.conf to /usr/local/etc (chmod 644)
- Copy digitalocean.sh to /usr/local/sbin (chmod 700)
- Add `digitalocean="YES"` to /etc/rc.conf
- Edit /usr/local/etc/digitalocean.conf if needed (self-documenting)

## Testing
- After installation, test it by entering `service digitalocean start`
- This builds configuration files but does not change any active network settings
- If no errors, check the `hostname`, `network`, and `routing` files in /etc/rc.conf.d
- If you configured a `droplet_user` (typically *freebsd*), check its home directory for .ssh/authorized_keys
- If all looks good, you can restart the droplet

## Activation
- Restart the droplet to rebuild the configuration files from the droplet's current metadata.
- FreeBSD then continues with its normal network initialization.

## Zero downtime network updates

It is possible to reconfigure the FreeBSD instance while running. Then restart networking and routing without needing to reboot:

```
# /usr/local/sbin/digitalocean.sh
# /etc/rc.d/netif restart && /etc/rd.d/routing restart
```

## Why I made this
As of the time of this writing, DigitalOcean's FreeBSD base images did not support ZFS. In order to get that feature, you had to roll your own FreeBSD installation following these handy steps (see https://github.com/fxlv/docs/blob/master/freebsd/freebsd-with-zfs-digitalocean.md).

In addition to gaining ZFS, you also get a completely stock, pristine FreeBSD system with nothing weird added to it. No extra packages, no extra users, and no mysterious files or directories added by the fine folks at DigitalOcean. Awesome as they are, we really don't want anyone adding junk to our droplets.

Of course, this also means the droplet requires the old-school network configurations in /etc/rc.conf, just like we had to do back in the day when our beards weren't so gray. But the downside of that means every new droplet you create from snapshot images boots up with hard-coded, conflicting network settings, and that's bad.

To get a droplet to determine its own configuration from the DigitalOcean assigned metadata, we call upon their API to grab all the key settings. That's what this script does at boot time, which also means it gets the latest metadata. Now you can shutdown a droplet, change its settings, and power it on for proper operation without needing to edit or change anything inside the FreeBSD instance. And that's good.
