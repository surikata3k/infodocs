# VMware Workstation Pro: network types

VMware Workstation Pro gives every virtual network adapter a **network connection type**. It decides who the VM can talk to: your physical LAN, the internet, only the host, or only other VMs.

You set it per adapter in **VM → Settings → Network Adapter**. A VM can have several adapters, each one connected to a different network.

## Overview

| Type | Virtual switch | VM gets its IP from | VM ↔ Internet | VM ↔ Host | VM ↔ Physical LAN | Typical use |
|---|---|---|---|---|---|---|
| **Bridged** | VMnet0 | Your real router (LAN DHCP) | ✅ | ✅ | ✅ Visible as a real machine | Servers other devices must reach |
| **NAT** | VMnet8 | VMware DHCP | ✅ Through the host | ✅ | ❌ Only outbound (or port forwarding) | Default: internet access, isolated from the LAN |
| **Host-only** | VMnet1 | VMware DHCP | ❌ | ✅ | ❌ | Private lab between host and VMs |
| **Custom** | VMnet2–VMnet19 | Your choice | Depends on config | Optional | Optional | Labs with several separate subnets |
| **LAN segment** | — (no switch on the host) | Nobody (static IPs or your own DHCP server) | ❌ | ❌ | ❌ | VM-only networks (firewall/router labs) |

## Bridged (VMnet0)

The VM connects **directly to your physical network**, as if it were another computer plugged into the same switch or Wi-Fi.

- It gets an IP from your real router, for example `192.168.1.x`, the same range as your PC.
- Any device on the LAN can reach it, and it can reach them.
- It's the easiest option when you want to connect by SSH from another computer, or run a server that others use.

!!! note "Laptops and Wi-Fi"
    - Tick **Replicate physical network connection state** if the laptop moves between networks. The VM then renews its IP when you change Wi-Fi.
    - In the Virtual Network Editor, VMnet0 is set to **Automatic** by default. If bridging picks the wrong card, choose the physical adapter manually (Ethernet or Wi-Fi).
    - Some networks (school, corporate, eduroam) only allow one MAC address per port or block unknown devices. In that case Bridged may not get an IP, so use NAT instead.

## NAT (VMnet8)

The VM is on a **private network behind the host**. The host does address translation, like a home router.

- VMware's DHCP gives the VM an IP on the VMnet8 subnet, for example `192.168.80.x`. The subnet is chosen randomly when VMware is installed.
- The VM has internet access through the host's connection, whatever that connection is: Wi-Fi, Ethernet or VPN.
- The host can reach the VM. Other machines on the LAN **cannot**, unless you set up port forwarding.
- This is the default type for new VMs, and it works on almost any network.

Default addresses on the VMnet8 subnet (`x` = your subnet):

| Address | Role |
|---|---|
| `192.168.x.1` | Host's virtual adapter (*VMware Network Adapter VMnet8*) |
| `192.168.x.2` | NAT gateway and DNS: this is the VM's default gateway |
| `192.168.x.128`–`.254` | DHCP range |

### Port forwarding (NAT)

To reach a NAT VM from another computer, forward a port on the host to the VM:

1. Open **Edit → Virtual Network Editor** and click **Change Settings** (it needs admin rights).
2. Select **VMnet8 → NAT Settings… → Add…**
3. Example for SSH: Host port `2222`, type `TCP`, VM IP `192.168.x.130`, VM port `22`.

Then from any machine on the LAN:

```bash
ssh -p 2222 user@<host-IP>
```

!!! tip
    Give the VM a fixed IP, or a DHCP reservation, so the forwarding rule doesn't break when its address changes.

## Host-only (VMnet1)

A **private network between the host and the VMs**, with no way out to the internet or the LAN.

- VMware's DHCP gives the VM an IP on the VMnet1 subnet. The host has its own address there through *VMware Network Adapter VMnet1*.
- The host and all VMs on VMnet1 can talk to each other.
- It's useful for isolated labs, or for testing services you don't want exposed.

A common pattern is to give a VM **two adapters**: one on **NAT**, for updates and internet, and one on **Host-only**, for a stable management IP that the host uses for SSH or the web.

## Custom (VMnet2–VMnet19)

These are extra virtual switches that you create in the **Virtual Network Editor**. Each one can be configured as:

- **Host-only**: with or without VMware DHCP, and with or without connecting the host's virtual adapter.
- **Bridged**: to a specific physical adapter, for example one VMnet bridged to Ethernet and another to Wi-Fi.
- **NAT**: only **one** NAT network is allowed at a time, and it's normally VMnet8.

They're useful for building labs with several subnets, for example:

```
VMnet2  10.0.10.0/24   → "LAN" network
VMnet3  10.0.20.0/24   → "DMZ" network
```

If you **untick** *Connect a host virtual adapter to this network* and *Use local DHCP service*, the VMnet works as an isolated switch. Only the VMs connected to it can see each other, which is very similar to a LAN segment.

## LAN segments

A LAN segment is a **private virtual switch only for VMs**. The host has no adapter on it and VMware provides no DHCP.

- You create them per VM: **VM Settings → Network Adapter → LAN segment → LAN Segments… → Add**.
- VMs connected to the same segment see each other. Nothing else can.
- You must use **static IPs**, or run your own DHCP server inside a VM (pfSense, Windows Server, `isc-dhcp-server`, etc.).

They're perfect for networking and security labs:

```
           Internet
              │
         [ NAT VMnet8 ]
              │ WAN
        ┌─────────────┐
        │ pfSense VM  │  ← router/firewall with 2 adapters
        └─────────────┘
              │ LAN
     [ LAN segment "LAN-Lab" ]
        │            │
    Ubuntu VM    Windows VM   ← only reach the internet through pfSense
```

## The Virtual Network Editor

Open it from **Edit → Virtual Network Editor** and click **Change Settings** to edit (it needs admin rights). From there you can:

- Change the subnet and mask of each VMnet.
- Turn VMware's DHCP on or off, and set its range (**DHCP Settings…**).
- Configure the NAT gateway and port forwarding (**NAT Settings…**).
- Choose which physical adapter VMnet0 bridges to.
- Add or remove custom VMnets.
- **Restore Defaults**: rebuilds all virtual networks if something breaks. This deletes custom VMnets and NAT rules.

## Which one should I use?

| I want… | Use |
|---|---|
| The VM to have internet without any fuss | **NAT** |
| Other computers on my network to reach the VM | **Bridged** (or NAT + port forwarding) |
| To reach the VM from my host only, with a stable IP | **Host-only** (often plus a NAT adapter) |
| Several isolated subnets for a lab | **Custom VMnets** or **LAN segments** |
| A router/firewall VM between an "internal" network and the internet | WAN on **NAT** + LAN on a **LAN segment** |

## Useful commands inside the VM

```bash
# Linux
ip a                 # IP addresses
ip r                 # routes / default gateway
resolvectl status    # DNS (Ubuntu)
sudo dhclient -r && sudo dhclient   # renew DHCP lease (if dhclient is installed)
```

```powershell
# Windows
ipconfig /all
ipconfig /release
ipconfig /renew
```

On the Windows host, `ipconfig` shows the **VMware Network Adapter VMnet1** and **VMnet8** adapters with the host's IP on each network.
