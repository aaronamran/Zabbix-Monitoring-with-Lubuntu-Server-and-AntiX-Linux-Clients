# Zabbix Monitoring with Lubuntu Server and antiX Linux Clients

This write-up documents a practical monitoring project using VirtualBox to simulate a lightweight infrastructure monitoring environment with Zabbix. A Lubuntu VM is configured as the Zabbix server, while antiX Linux VMs act as monitored clients. This setup demonstrates how to efficiently monitor system resource usage (CPU, RAM, disk) in a minimal-resource lab, ideal for single-laptop environments. The focus is on setting up agent-based monitoring, configuring basic alerts, and optimizing performance without relying on heavy operating systems or cloud infrastructure. The antiX Linux VM setup in VirtualBox is included at the beginning of this write-up, while the initial Lubuntu VM setup in VirtualBox is not included as it was completed beforehand.


1. [antiX Linux VM Setup in VirtualBox](https://github.com/aaronamran/Zabbix-Monitoring-with-Lubuntu-Server-and-AntiX-Linux-Clients/blob/main/README.md#antix-linux-vm-setup-in-virtualbox)
2. [Monitoring Server VM Setup](https://github.com/aaronamran/Zabbix-Monitoring-with-Lubuntu-Server-and-AntiX-Linux-Clients/blob/main/README.md#monitoring-server-vm-setup)
3. [Client VM Setup](https://github.com/aaronamran/Zabbix-Monitoring-with-Lubuntu-Server-and-AntiX-Linux-Clients/blob/main/README.md#client-vm-setup)
4. [Adding Hosts and Setting Up Alerts](https://github.com/aaronamran/Zabbix-Monitoring-with-Lubuntu-Server-and-AntiX-Linux-Clients/blob/main/README.md#adding-hosts-and-setting-up-alerts)
5. [Testing Monitoring and Visualisation](https://github.com/aaronamran/Zabbix-Monitoring-with-Lubuntu-Server-and-AntiX-Linux-Clients/blob/main/README.md#testing-monitoring-and-visualisation)


## antiX Linux VM Setup in VirtualBox
antiX Linux has two possible init systems, which are the sysVinit (default option) or runit. For this homelab project, sysVinit which is the default option is used. Aditionally, each of these architectures and init system comes in four possible "flavours", which are:
- antiX-full (c1.8GB) - 4 windows managers – IceWM (default), fluxbox, jwm and herbstluftwm plus full libreoffice suite. Suitable for most users. Lots of applications pre-installed and has the best hardware support.
- antiX-base (c1.2GB) – 4 windows managers – IceWM (default), fluxbox, jwm and herbstluftwm. Suitable for those who want to customise what to install.
- antiX-core (c520MB) – no gui environment, but should support most wireless. Suitable for confident/comfortable users who want to build up from a minimal install.
- antiX-net (c220MB)- no X. Just enough to get you connected (wired) and ready to build. (NOTE – to connect to the Internet, you might need to type as root user ifup eth0 or ifup eth1). Suitable for Advanced users.




## Monitoring Server VM Setup



## Client VM Setup



## Adding Hosts and Setting Up Alerts



## Testing Monitoring and Visualisation
