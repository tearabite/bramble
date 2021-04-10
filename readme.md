# Set Up a Raspberry Pi Cluster
## Master Setup
1. Write the Ubuntu Server Image to All SD Cards
1. Use Raspberry Pi Imager.
1. Open the file `/boot/cmdline.txt'.
 - Add `cgroup_enable=memory cgroup_memory=1` to the beginning of the file.
 - This can be done after booting the Pi, if desired, by editing `/boot/firmware/cmdline.txt'
1. Use this guide to set up WiFi and eth1 https://itsfoss.com/connect-wifi-terminal-ubuntu/
1. Install and configure `dhcpcd5`
 - `sudo apt install dhcpcd5`
 - `sudo vim /etc/dhcpcd.conf`
 - ```
    interface eth0
    static ip_address=10.0.0.1/8
    static domain_name_servers=8.8.8.8,8.8.4.4
    nolink
    ```
1. Install and configure `dnsmasq`
 - `sudo apt install dnsmasq`
 - `sudo vim /etc/dnsmasq.conf`
 - Use the dnsmasq.conf supplied in the git repo
1. Set the hostname
- `sudo hostnamectl set-hostname bramble-master`
1. `sudo vim /etc/init.d/dnsmasq
 - ```
   # Hack to wait until dhcpcd is ready
   sleep 10
   ```
1. `sudo vim /etc/sysctl.conf`
 - Uncomment `net.ipv4.ip_forward=1`
1. Forward internet traffic through wlan0 or eth1 into eth0
 - ```
    sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
    sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
    ```
1. `sudo apt install iptables-persistent`

## Worker Setup
1. `sudo hostnamectl set-hostname bramble-worker-#`
1. `sudo vim /boot/firmware/cmdline.txt`
 - `cgroup_enable=memory cgroup_memory=1`
1. Disable swap

## microk8s Setup
1. `sudo snap install microk8s --classic`
1. Run the commands to add my user to the sudoers group
1. Add nodes

1. `microk8s enable prometheus`
1. Add ingress with `sudo microk8s kubectl apply -f grafana.yaml`
1. Patch grafana with `sudo microk8s kubectl patch deployment grafana -n monitoring --patch "$(cat grafana_patch.yaml)"`
1. Patch prometheus with `` 