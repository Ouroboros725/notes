Solution:
```
$ crc delete
$ sudo iptables -F
$ sudo systemctl restart firewalld 
$ sudo systemctl restart libvirtd 
$ crc setup 
```

https://github.com/code-ready/crc/issues/712

https://github.com/code-ready/crc/issues/494


I started seeing this today.

After a lot of digging I noticed on boot an error in the journal:
"May 26 11:19:08 jmontleo.usersys.redhat.com firewalld[2850]: ERROR: ZONE_CONFLICT: 'docker0' already bound to a zone"

This ended up being because I used the workaround of adding docker0 to the trusted zone for:
https://bugzilla.redhat.com/show_bug.cgi?id=1817022

It looks like a new moby-engine package went to stable a few days ago which added a docker.xml zone file in /usr/lib/firewalld/zones.

I ended up having to delete /etc/firewalld/zones/trusted.xml to remove the workaround. One this was done and I rebooted libvirt (and my ethernet interface for that matter!) ended up in the correct zones.

It is not great that firewalld stops adding interfaces to zones and becomes pretty unusable when it hits a zone conflict like that. Might be better to move this to firewalld to see if they can do anything about that.

https://bugzilla.redhat.com/show_bug.cgi?id=1829090