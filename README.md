# Installing-kvm&sushy-tool-on-rhel

## Install KVM and Virtual Machine Manager (virt-manager)

Step 1. Check CPU Virtualization Support
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```
If output ≥ 1 → Virtualization supported.

Step 2. Enable Virtualization Packages

Make sure RHEL repositories are enabled and system is updated:
```bash
sudo dnf update -y
sudo dnf install -y @virtualization
```
This will install qemu-kvm, libvirt, virt-install, virt-manager, virt-viewer

Step 3. Start and Enable libvirtd
```bash
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
```
Step 4. Verify Installation
```bash
virsh list --all
```
If it lists successfully, KVM is working

Step 5. Install Graphical Virtual Machine Manager 
```bash
sudo dnf install virt-manager -y
```
Then launch with:
```bash
virt-manager
```
You will get a graphical interface of KVM

## Install sushy / sushy-tools (Redfish simulator)

Step 1. Install Python and Dependencies
```bash
sudo dnf install -y python3 python3-pip
```

Step 2. Install sushy-tools
```bash
pip3 install sushy-tools
```
Step 3. Configure sushy-tools (for virtualbmc or libvirt)
```bash
mkdir -p ~/.config/sushy-tools
vim ~/.config/sushy-tools.conf
```
Then add below lines in file: 
```
[SushyEmulator]
debug = true
auth_file = ~/.config/sushy-tools/auth.json
libvirt_uri = qemu:///system
```
Then start it:
```bash 
sushy-emulator
```

You’ll get a running Redfish API service that interfaces with your libvirt/KVM environment.

Try This in Browser:

Open this URL:
```
http://localhost:8000/redfish/v1/
```

**Next Steps**

Step 1. Check the systems managed by libvirt/ list of all Vms
```
http://localhost:8000/redfish/v1/Systems
```
Step 2. Verify managers (virtual BMC endpoints)/ List all virtual managers
```
http://localhost:8000/redfish/v1/Managers
```
Step 3. See a specific system (replace <vm-name> with actual VM)
```
http://localhost:8000/redfish/v1/Systems/<vm-name>
```

**CLI MODE**

List Vms Using Curl
```bash
curl http://localhost:8000/redfish/v1/Systems | jq
```


## Control VM Power State via Redfish(replace <vm-name> with actual VM)
Power On:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"ResetType": "On"}' http://localhost:8000/redfish/v1/Systems/<vm-name>/Actions/ComputerSystem.Reset
````
Power Off:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"ResetType": "ForceOff"}' http://localhost:8000/redfish/v1/Systems/<vm-name>/Actions/ComputerSystem.Reset
````
Restart VM:
```bash 
curl -X POST -H "Content-Type: application/json" -d '{"ResetType": "GracefulRestart"}' http://localhost:8000/redfish/v1/Systems/<vm-name>/Actions/ComputerSystem.Reset
```
