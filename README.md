# Cisco IOS XE 3.17 running Ubuntu Image

## Network Hosted Kernel Virtual Machine (KVM) on Cisco IOS XE  
  
Many Cisco Devices allow you to host your own industry-standard KVM virtual machine directly in the network.  
Whether you're a for-profit Cisco partner, Open Source project, Educator or End User, this page will get you started crafting your own application.  
  
## More information at [https://developer.cisco.com/site/kvm/](https://developer.cisco.com/site/kvm/)

## Prepacked

```bash
git clone https://github.com/robertcsapo/cisco-ios-xe-ubuntu.git
```  


## Package your own image

```bash
git clone https://github.com/robertcsapo/cisco-ios-xe-ubuntu.git
tar -xvf ubuntu_cloud.ova
rm -rf *.qcow2 *.ver *.mf *.ova
wget https://your_image.img
nano package.yaml
echo "1.0" > version.ver
cp your_image.img to your_image.qcow2
qemu-img resize your_image.qcow2 +1G (or whatever size you want your disk)
openssl sha1 *.qcow2 *.ver *.yaml > your_image.mf
tar -cvf your_image.ova *.qcow2 *.ver *.yaml *.mf
```


## Login to Cisco IOS XE

```
copy scp: flash:  
virtual-service install name ubuntu_cloud package flash:ubuntu_cloud.ova  
show virtual-service list  

Virtual Service List:


Name                    Status             Package Name                         
------------------------------------------------------------------------------
csr_mgmt                Installed          iosxe-remote-mgmt.03.17.00.S.156...  
ubuntu_cloud            Installed          ubuntu_cloud.ova                     

```

### Configure Cisco IOX XE

### First VirtualPortGroup interface

```
interface VirtualPortGroup0
 ip address 10.0.0.1 255.255.255.0
 ip nat inside
 no mop enabled
 no mop sysid
end
```

### Addning DHCP for Ubuntu (Optional)

```
ip dhcp pool containers-dhcp
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1 
 dns-server 8.8.8.8 
```

### Adding NAT for Ubuntu (Optional)

```
access-list 10 permit 10.0.0.0 0.0.0.255
!
interface VirtualPortGroup0
 ip nat inside
!
interface GigabitEthernet1
 ip nat inside
!
ip nat inside source list 10 interface GigabitEthernet1 overload
```

### Linking container to VirtualPortGroup interface

```
virtual-service ubuntu_cloud
 vnic gateway VirtualPortGroup0
 activate
!
```

### Connect to Ubuntu Console (wait a couple of minute, depending on your CPU load of the router)

```
virtual-service connect name ubuntu_cloud console
Connected to appliance. Exit using ^c^c^c
```

# Notice:
  
## Use for Proof of Concept (with the ubuntu_cloud.ova provided)

> Login: ubuntu/cisco  
>  
> Disabled cloud connecting for fast boot  
> /etc/cloud/cloud.cfg.d/90_dpkg.cfg

## Issues  
  
* I can take some time to clone/download this repo, as the prepacked .ova is included in the master.  
* Increase memory (512MB is not enough) if you get ```fatal: Out of memory, malloc failed```