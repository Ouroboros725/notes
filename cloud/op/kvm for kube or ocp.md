https://docs.okd.io/latest/minishift/getting-started/setting-up-virtualization-environment.html

On Fedora
1. Install `libvirt` and `qemu-kvm` on your system:
```
$ sudo dnf install libvirt qemu-kvm
```
2. Add yourself to the `libvirt` group:
```
$ sudo usermod -a -G libvirt $(whoami)
```
3. Update your current session to apply the group change:
```
$ newgrp libvirt
```
4. As root, install the KVM driver binary and make it executable as follows:
```
# curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-centos7 -o /usr/local/bin/docker-machine-driver-kvm
# chmod +x /usr/local/bin/docker-machine-driver-kvm
```

---

https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm2-driver

Fedora/CentOS/RHEL:
```
sudo yum install libvirt-daemon-kvm qemu-kvm
```
Enable,start, and verify the libvirtd service has started.
```
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
sudo systemctl status libvirtd.service
```
Then you will need to add yourself to libvirt group (older distributions may use libvirtd instead)
```
sudo usermod -a -G libvirt $(whoami)
```
Then to join the group with your current user session:
```
newgrp libvirt
```
Now install the driver:
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2 \
  && sudo install docker-machine-driver-kvm2 /usr/local/bin/
  ```