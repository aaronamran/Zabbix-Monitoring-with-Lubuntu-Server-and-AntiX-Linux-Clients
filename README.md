# Zabbix Monitoring with Lubuntu Server and antiX Linux Clients

This write-up documents a practical monitoring project using VirtualBox to simulate a lightweight infrastructure monitoring environment with Zabbix. A Lubuntu VM is configured as the Zabbix server, while antiX Linux VMs act as monitored clients. This setup demonstrates how to efficiently monitor system resource usage (CPU, RAM, disk) in a minimal-resource lab, ideal for single-laptop environments. The focus is on setting up agent-based monitoring, configuring basic alerts, and optimizing performance without relying on heavy operating systems or cloud infrastructure. The antiX Linux VM setup in VirtualBox is included at the beginning of this write-up, while the initial Lubuntu VM setup in VirtualBox is not included as it was completed beforehand.


1. [antiX Linux VM Setup in VirtualBox](#antix-linux-vm-setup-in-virtualbox)
2. [Monitoring Server VM Setup](#monitoring-server-vm-setup)
3. [Client VM Setup and Agent Testing](#client-vm-setup-and-agent-testing)
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
  ![image](https://github.com/user-attachments/assets/b64b7202-0301-4c5d-ae49-adabaa06a2a0)
  <br />
  However, as seen in the screenshot below, an error with unmet dependencies for Zabbix was encountered. The version of Lubuntu VM at this point in time is Ubuntu 24.04 (Noble Numbat) which is very new, and Zabbix has not released official packages for it yet <br />
  ![image](https://github.com/user-attachments/assets/f51eb3b0-712d-4ee8-9a1f-ec6045e38ba1)
  <br />
  A workaround for this error is to run each of the command below to download the Zabbix 6.0 repo for Ubuntu 22.04 (Jammy)
  ```
  wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
  sudo dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
  sudo apt update
  ```
  Ubuntu 24.04 uses `libldap-2.6-0`, but Zabbix 6.0 depends on `2.5`. So the `.deb` file from Jammy (22.04) is grabbed and installed side-by-side. The download link can be obtained from [http://archive.ubuntu.com/ubuntu/pool/main/o/openldap/](http://archive.ubuntu.com/ubuntu/pool/main/o/openldap/)
  ```
  wget http://archive.ubuntu.com/ubuntu/pool/main/o/openldap/libldap-2.5-0_2.5.18+dfsg-0ubuntu0.22.04.3_amd64.deb
  sudo dpkg -i libldap-2.5-0_2.5.18+dfsg-0ubuntu0.22.04.3_amd64.deb
  ```
  ![image](https://github.com/user-attachments/assets/3ffd19ce-84a7-4d4c-bfbf-e1e4409fa0bb)
  <br />
  Continue installing the Zabbix server, frontend, agent and MariaDB
  ```
  sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mariadb-server -y
  ```
  In case dependency issues still occur, use
  ```
  sudo apt --fix-broken install
  ```
  
- Verify server and agent after installation
  ```
  zabbix_server -V
  zabbix_agentd -V
  ```
  ![image](https://github.com/user-attachments/assets/c4042c34-928f-47fa-accf-115b01c00c20)


- To configure MySQL (MariaDB), secure MySQL and create the Zabbix Database
  ```
  sudo mysql_secure_installation
  sudo mysql -uroot -p
  ```
  ![image](https://github.com/user-attachments/assets/cc677b2f-7285-4031-8fd0-a67d3b275a70)

- In the MySQL prompt, run
  ```  
  CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_bin;
  CREATE USER zabbix@localhost IDENTIFIED BY 'zabbixpass';
  GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
  FLUSH PRIVILEGES;
  EXIT;
  ```
  ![image](https://github.com/user-attachments/assets/d9a184b8-ab3a-4813-8ebf-3117b6e45850)

- Import the DB Schema
  ```
  zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
  ```
  ![image](https://github.com/user-attachments/assets/fbfa228a-0534-4baa-853e-ace4fe7a1e03)
  <br />
  Note that if `zabbixpass` is not used as the password, it will return ERROR 1045 (28000) as shown in the screenshot
- Edit the Zabbix config
  ```
  sudo nano /etc/zabbix/zabbix_server.conf
  ```
  and set
  ```
  DBPassword=zabbixpass
  ```
  ![image](https://github.com/user-attachments/assets/deb2375d-e81c-43a7-be88-6703cbdd4c2d)

- Start the services
  ```
  sudo systemctl restart zabbix-server zabbix-agent apache2
  sudo systemctl enable zabbix-server zabbix-agent apache2
  ```
- To access the web UI, open a web browser like Firefox and enter `http://<Lubuntu_IP_address>/zabbix`. On first time installation, it will redirect us to `http://<Lubuntu_IP_address>/zabbix/setup.php`. Choose the default language and click Next step<br />
  ![image](https://github.com/user-attachments/assets/3936bb27-32e6-42fa-a772-7c7adb6e3e4d)
  <br />
  For the Check of pre-requisites page, ensure that all items listed are OK
  <br />
  ![image](https://github.com/user-attachments/assets/afc8f33c-baa5-4599-ab0d-741cee2aa508)
  <br />
  In the Configure DB connection, key in the password `zabbixpass` that we created and used in the earlier steps
  <br />
  ![image](https://github.com/user-attachments/assets/007e3716-9288-44b1-b1ef-820a3504a64e)
  <br />
  Next in the Settings page, give a name to the server and choose the correct time zone
  <br />
  ![image](https://github.com/user-attachments/assets/c02fd5c4-5a70-49a0-8d63-2c5dd0a4e95a)
  <br />
  A Pre-installation summary page will appear next before proceeding with the installation
  <br />
  ![image](https://github.com/user-attachments/assets/e78211fd-b8cc-4d1f-87c5-cae991e62921)
  <br />
  The following message will appear upon successful configuration and installation
  <br />
  ![image](https://github.com/user-attachments/assets/c5751cdd-2b6c-4ae0-b867-793f81121306)



## Client VM Setup and Agent Testing
- After starting antiX-core Linux VM for the first time, login as root with the password root. Then run the following command to install antiX-core on the virtual hard drive
  ```
  cli-installer
  ```
  The following prompt will appear, enter y for yes
  <br />
  ![image](https://github.com/user-attachments/assets/3eb8f640-e1f2-413f-ba0b-5e867d62f60b)

- A label type has to be selected. `gpt` which stands for GUID partition table is recommended for modern systems especially if a UEFI VM setup is used and the disk is larger than 2TB. Based on this homelab project, `dos` will be used due to our VM settings for smaller disks
  ![image](https://github.com/user-attachments/assets/6c7c5ada-7708-4f64-ae8c-3d89ba19804c)
  <br />
  Then press Enter to create a new partition from free space and proceed with partition size of 2GB
  <br />
  ![image](https://github.com/user-attachments/assets/6e881446-a770-4d31-9e22-685cea621ce6)
  <br />
  `sda` is used as the root partition. If you are unsure what the partition is named, quit the installer and run the command `lsblk`, then restart the entire process until this installation step is reached
  <br />
  ![image](https://github.com/user-attachments/assets/d7d294a0-fbcf-40db-b9f9-fe98dc95bbab)
- In terms of choosing a filesystem, the most compatible choice for this project is `ext4`. After choosing `ext4`, the installer will format `/dev/sda` with ext4 and install antiX-core. Then reboot the VM
  ![image](https://github.com/user-attachments/assets/8a338e40-3b4e-461e-b6ad-ad0b21a48408)






- Since antiX-core Linux VM has been installed earlier, install Zabbix Agent
  ```
  sudo apt update
  sudo apt install zabbix-agent -y
  ```
- Then configure Zabbix Agent
  ```
  sudo nano /etc/zabbix/zabbix_agentd.conf
  ```
  And set the following line to point to your Zabbix server
  ```
  Server=<IP of Lubuntu VM>
  ```
  ![image](https://github.com/user-attachments/assets/24c444a8-62b7-4359-a99d-c6b7efc6ea59)

- Start the Agent service
  ```
  sudo service restart zabbix-agent
  sudo service enable zabbix-agent
  ```
  If the command does not work or is not recognised, use the format of the next commands. <br />
  To check if the service is running, use
  ```
  sudo /etc/init.d/zabbix-agent status
  ```
  Likewise, to restart the service, use
  ```
  sudo /etc/init.d/zabbix-agent restart
  ```
  ![image](https://github.com/user-attachments/assets/76bff0b1-573a-4fd8-9a94-33190f7da16e)
- Clone the current client VM so that the configurations are the standardised





## Adding Hosts and Setting Up Alerts
- Ensure that the previous config files such as `/etc/zabbix/zabbix_server.conf` are modified correctly

- Log into Zabbix Web UI on Lubuntu Server VM. The default username is Admin and password is zabbix
  ![image](https://github.com/user-attachments/assets/c4728e8d-bebd-4d30-a0f3-250b9beafbac)

- Before setting up triggers and dashboards, test whether the Zabbix Server can successfully communicate with the client. From the Zabbix Server, run
  ```
  zabbix_get -s <Client_IP> -k system.uptime
  ```
  ![image](https://github.com/user-attachments/assets/2260c57c-9c33-4a12-a7c0-fcecf054126d)

  We should get a number that represents the system uptime in seconds. If a response is not received, check your network connectivity, firewall settings, and the `zabbix_agentd.conf` settings. In the screenshot, it shows that there was access restriction in Zabbix agent configuration. This is due to ServerActive in `zabbix_agentd.conf` not being set solely to be the Lubuntu Server VM's IP address. At this point of the homelab project, the original antiX-core Linux VM should already be cloned. If any processes are running on it, the machine's state can be saved first, then exited and cloned.
- When both client VMs are powered on, check their IP address. If both VMs have the same IP address, it is due to them having the same MAC address, which causes the DHCP server to assign an identical IP. The solution for this in VirtualBox is to navigate to the Network settings of the cloned VM, and regenerate a new MAC address.
  ![image](https://github.com/user-attachments/assets/674e5a11-bdc5-471d-a447-ff537d69b44d)
- For the cloned client VM, make sure that it has Zabbix agent installed. If it is not installed, repeat the previous steps for the original VM to ensure both VMs are on the same level of progress before proceeding to be added as hosts in Lubuntu Server VM
- Go to `Configuration > Hosts > Create Host` and set the following:
  - Hostname: Match /etc/hostname of client VM
  - IP: Client VM’s IP address
  - Group: Linux Servers
  - Template: Template OS Linux
  ![image](https://github.com/user-attachments/assets/56d5e798-f46c-4328-8173-f1d9a4c07dd0)

- To set up alerts for high CPU usage, go to `Configuration > Actions > Trigger actions` and create a new action:
  - Name: High CPU usage alert
  - Condition: Trigger name = "Processor load is too high on {HOST.NAME}"
  - Operation: Send message to a
  ![image](https://github.com/user-attachments/assets/e80d9b42-9eef-472f-bdc4-66a501b99698)

- To create an alert for low disk space, navigate to `Configuration > Hosts > [Select your host] > Items`
  ![image](https://github.com/user-attachments/assets/dbaed33c-3a53-45c2-b1d2-955928021ef6)

- Create an item with the following parameters then save it:
  - Name: Free disk space on /
  - Type: Zabbix agent
  - Key: vfs.fs.size[/,free]
  - Type of information: Numeric (unsigned)
  - Units: B (bytes)
  - Update interval: 60s
  ![image](https://github.com/user-attachments/assets/3e4fdade-8f9c-454f-a65e-ec97f8323538)

- After creating a trigger for low disk space, switch to the Triggers tab and add:
  - Name: Low disk space on /
  - Expression: {<YourHost>:vfs.fs.size[/,free].last()}<500000000
  - Severity: Warning
  ![image](https://github.com/user-attachments/assets/d6e9ff25-fe1d-4c3f-828c-f1a6c277b3c7)

- To create alerts for unreachable hosts, first verify the host availability by navigating to `Configuration > Hosts` and ensuring the host's availability is enabled (look for green or gray dot next to Agent/ICMP)
- Zabbix will automatically trigger and alert if the agent stops responding. If needed, the detection interval can be adjusted via `Administration > General > Housekeeping > Availability`



## Testing Monitoring and Visualisation
- To view graphs in the Zabbix web UI, go to `Monitoring > Hosts > [Your Host] > Graphs`
  ![image](https://github.com/user-attachments/assets/10a0a7a7-abcd-49c6-82de-bd2de9a3a331)

- Create dashboards in `Monitoring > Dashboard > Create`. New dashboards are created by adding widgets
  ![image](https://github.com/user-attachments/assets/df6a1574-9a03-4238-9a31-f97bac462608)

- The global view dashboard looks like the following
  ![image](https://github.com/user-attachments/assets/01252637-5dd7-4ad2-a2e1-0294cbef3ade)
 
- To create a specific dashboard for stress testing CPUs, add a graph as widget, give it a title, choose `antix1` as data set, and `CPU utilisation` as item. Then reposition the widget to a suitable place on screen and save the changes
  ![image](https://github.com/user-attachments/assets/881fcf73-54e8-42bf-861c-0f58fcbd8d26)

- To stress test the client VM, run the following command in antiX-core Linux VM
  ```
  sudo apt install stress
  stress --cpu 2 --timeout 60
  ```
  A temporary CPU spike should be visible in the Zabbix dashboard
  <br />
  ![image](https://github.com/user-attachments/assets/f5926e45-10c3-4346-808e-e5a6671aacd1)


