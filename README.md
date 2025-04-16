# Zabbix Monitoring with Lubuntu Server and antiX Linux Clients

This write-up documents a practical monitoring project using VirtualBox to simulate a lightweight infrastructure monitoring environment with Zabbix. A Lubuntu VM is configured as the Zabbix server, while antiX Linux VMs act as monitored clients. This setup demonstrates how to efficiently monitor system resource usage (CPU, RAM, disk) in a minimal-resource lab, ideal for single-laptop environments. The focus is on setting up agent-based monitoring, configuring basic alerts, and optimizing performance without relying on heavy operating systems or cloud infrastructure. The antiX Linux VM setup in VirtualBox is included at the beginning of this write-up, while the initial Lubuntu VM setup in VirtualBox is not included as it was completed beforehand.


1. [antiX Linux VM Setup in VirtualBox](#antix-linux-vm-setup-in-virtualbox)
2. [Monitoring Server VM Setup](#monitoring-server-vm-setup)
3. [Client VM Setup](#client-vm-setup)
4. [Adding Hosts and Setting Up Alerts](#adding-hosts-and-setting-up-alerts)
5. [Testing Monitoring and Visualisation](#testing-monitoring-and-visualisation)


## antiX Linux VM Setup in VirtualBox
antiX Linux has two possible init systems, which are the sysVinit (default option) or runit. Additionally, each of these architectures and init system comes in four possible "flavours", which are:
- antiX-full (c1.8GB) - 4 windows managers – IceWM (default), fluxbox, jwm and herbstluftwm plus full libreoffice suite. Suitable for most users. Lots of applications pre-installed and has the best hardware support
- antiX-base (c1.2GB) - 4 windows managers – IceWM (default), fluxbox, jwm and herbstluftwm. Suitable for those who want to customise what to install
- antiX-core (c520MB) - no gui environment, but should support most wireless. Suitable for confident/comfortable users who want to build up from a minimal install
- antiX-net (c220MB) - no X. Just enough to get you connected (wired) and ready to build. (NOTE – to connect to the Internet, you might need to type as root user ifup eth0 or ifup eth1). Suitable for Advanced users

For this homelab project, sysVinit which is the default option is used. The specific flavour used is antiX-core (c520MB). 

- To get started, download the chosen ISO image of antiX Linux from [https://antixlinux.com/download/](https://antixlinux.com/download/)
- In VirtualBox, create the VM for antiX-core Linux
  ![image](https://github.com/user-attachments/assets/9661859d-4c58-41c5-a5c2-b5754eac39e7)
- Although antiX-core uses 192MB RAM per VM, let's just double the RAM needed. Set the number of processors to 1
  ![image](https://github.com/user-attachments/assets/ab602cd5-6678-4b13-a0f4-49687bbca828)
- Use 2GB of disk size as safe minimum per VM. After clicking Next, the VM setup is complete
  ![image](https://github.com/user-attachments/assets/84277727-d411-4faf-a5c4-6580b4a7e350)



## Monitoring Server VM Setup
- In the Lubuntu server VM, run the following commands to install Zabbix Server and Frontend
  ```
  wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
  sudo dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
  sudo apt update
  sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mariadb-server -y
  ```
- Then configure MySQL (MariaDB)
  ```
  sudo mysql_secure_installation
  sudo mysql -uroot -p
  CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_bin;
  CREATE USER zabbix@localhost IDENTIFIED BY 'zabbixpass';
  GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
  FLUSH PRIVILEGES;
  EXIT;
  ```
- Import the DB Schema
  ```
  zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
  ```
- Edit the Zabbix config
  ```
  sudo nano /etc/zabbix/zabbix_server.conf
  # Set:
  DBPassword=zabbixpass
  ```
- Start the services
  ```
  sudo systemctl restart zabbix-server zabbix-agent apache2
  sudo systemctl enable zabbix-server zabbix-agent apache2
  ```
- To access the web UI, open a web browser like Firefox and enter `http://<Lubuntu_IP_address>/zabbix`


## Client VM Setup
- Since antiX-core Linux VM has been installed earlier, install Zabbix Agent
  ```
  sudo apt update
  sudo apt install zabbix-agent -y
  ```
- Then configure Zabbix Agent
  ```
  sudo nano /etc/zabbix/zabbix_agentd.conf
  # Set:
  Server=<IP of Lubuntu VM>
  ```
- Start the Agent service
  ```
  sudo systemctl restart zabbix-agent
  sudo systemctl enable zabbix-agent
  ```
- Clone the current client VM so that the configurations are the same


## Adding Hosts and Setting Up Alerts
- Log into Zabbix Web Interface in Lubuntu Server VM
- Go to Configuration > Hosts > Create Host and set the following:
  - Hostname: Match /etc/hostname of VM
  - IP: VM’s IP address
  - Group: Linux Servers
  - Template: Template OS Linux
- To set up alerts, go to Configuration > Actions and create a new action:
  - Name: High CPU usage alert
  - Condition: Trigger name = "Processor load is too high on {HOST.NAME}"
  - Operation: Send message to admin
 


## Testing Monitoring and Visualisation
- Use built-in Zabbix Graphs in Monitoring > Hosts > [Your Host] > Graphs
- Create dashboards in Monitoring > Dashboard > Create
- To stress test a monitored client VM, run the following command in antiX-core Linux VM
  ```
  sudo apt install stress
  stress --cpu 2 --timeout 60
  ```
- A CPU usage spike should be visible in the Zabbix dashboard

