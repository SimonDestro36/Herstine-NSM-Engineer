# DAY 1  
---

__Pfsense Setup__  *Not Testable
1. Plug in Pfsense USB 2.0 PMAP 
2. Press F11 to go into setup and select UEFI 2.0 
    - Hit 5 and then enter if a blue screen does not pop up
3. Click Accept, and Install  
4. Leave it at Default for Keymap
5. Select UEFI Auto-Guided Route 
6. Choose No then Reboot  
7. Select 1 - Assign Interfaces
8. Choose No and Select Em0 and WAN 
9. Select Em1 and LAN 
10. Just hit enter and press Y
11. Select option 2, type number 2 again.
12. Set static IP to 172.16.50.1/24
13. Set LAN to same IP
14. Choose yes for DHCP
15. For the range 172.16.50.100 - 172.16.50.254
16. Pick Yes
17. Set your laptop IP to 172.16.50.200 and the DG to 172.16.50.1
18. Go to 172.16.50.1 via firefox for Web interface for Pfsense
---  
# DAY 2  
---

__Trouble-Shooting__  

- 192.168.2.1 --> Edge Router 
- 10.0.0.2 --> Downstream to Cisco Switch
- 10.0.0.1 --> Cisco Switch port 24 to Edge Router
- 10.0.50.1 --> Physical port on Switch 
- 172.16.50.1 --> Pfsense 
- 172.16.50.100 --> Sensor  
---  
__Sensor Setup__

1. Turn on Sensor, press F10 until the boot screen shows  
2. Select USB Part 1
3. Select test Centos 7 to make sure drive is good
4. Choose Continue, then select Network and Hostname
5. Management will be ENO1(Right Ethernet)
6. Once plugged in click it to on
7. Select Configure
8. IPv4 Setting-Method Manual 
9. Set 172.16.50.100-255.255.255.0-172.16.50.1  
10. Select IPv6-Method Ignore
11. Hit Save
12. Bottom Left corner put sg50.local.lan for hostname
13. Select Done then Date and Time
14. For Region select ETC and City: Coord. Uni. Time
15. Disable KDump
16. Select Installation Destination
17. Select both disks and choose Auto Conf. Parti.
18. Also select I would like to make addit. space avail.
19. Select Delete all and select Reclaim space
20. Go back into Install Dest.
21. Select I will Configure Partitioning
22. Hit Done
23. Select the blue link to create the partitions automatically
24. Choose Home then for desired capacity put 1 Gib
25. Select Update settings 
26. Click / and change that to 1 Gib 
27. Select Update Settings
28. Add a new mount /var/log and 1Gib for space 
29. Add another /tmp 1 G
30. Add another /var 1G
31. /data/stenographer500G//suricata25G//fsf10G//zeek25G//elasticsearch//kafka100G
32. /var/log/audit, /var/tmp
33. Highlight / and the click create new volume group 
34. Highlight the 500GB disk then rename it to vg_os
35. Click Update Settings
36. Do the same for /home, /tmp, /var, swap
37. Highlight /Data/Zeek and create new Volume Group named vg_data select the 1TB disk
38. Everything with /data will be in vg_data
39. Click Done and Accept changes
40. Click Begin installation
41. Create new user 
42. Click reboot 
43. Login 
44. Run a ip a command to determine if eno1 IP is right 
45. ssh into the sensor via student laptop
    - ssh admin@172.16.50.100  
46. Run 'sudo vi /etc/sysconfig/network-scripts/ifcfg-eno1'
47. Change all the IPv6 to "no" and ONBOOT to "yes"
48. wq! to save 
49. Enter sudo vi /etc/sysctl.conf
50. Type "net.ipv6.conf.all.disable_ipv6=1"
51. Type "net.ipv6.conf.default.disable_ipv6=1"
52. wq! to save 
53. Run 'sudo vi /etc/hosts'
54. Ignore the line ::1
55. sudo sysctl -p to restart  
---  

# Repository
---

- cd /etc/yum.repos.d
- ls -l
- sudo rm -rf CentOS-*
- Open up a browser and go to 192.168.2.20
- Go to share folder 
- sudo curl -LO http://192.168.2.20:8080/local.repo  
- sudo yum makecache  
- sudo yum list zeek
- sudo yum update / Y 
- sudo systemctl reboot
- sudo yum makecache --disablerepo="**" --enablerepo=local*
- sudo yum update --disablerepo="**" --enablerepo=local*
- sudo systemctl reboot  
---  

# SELinux  
---

Check status of SELinux: sestatus  

---  
# Sniffing Traffic  
---  

1. ip a
2. enp5s0 is used for sniffing traffic
3. sudo ethtool -k enp5s0 
4. sudo curl -LO http://192.168.2.20:8080/interface_prep.sh
5. ls -l
6. cat interface_prep.sh
7. sudo chmod +x interface_prep.sh
8. sudo ./interface_prep.sh enp5s0
9. sudo ethtool -k enp5s0 
10. cd /sbin
11. sudo curl -LO http://192.168.2.20:8080/ifup-local  
12. sudo chmod +x ifup-local
13. cd /etc/sysconfig/network-scripts/
14. sudo vi ifup and at the bottom add
    - if [ -x /sbin/ifup-local ]; then  
    /sbin/ifup-local ${DEVICE}  
    fi
15. Enter :wq
16. sudo vi ifcfg-enp5s0
17. BOOTPROTO = none, IPv6 = no, DEFROUTE = None, ONBOOT = yes, NM_CONTROLLED = no 
18. :wq to save
19. sudo systemctl reboot 
20. ssh admin@172.16.50.100
21. sudo ethtool -k enp5s0
22. keep checksum off
23. ip a 
24. enp5s0 should be promsc.
25. once the tap is in place ssh into the sensor
26. cd /etc/yum.repos.d 
27. sudo yum install tcpdump --disablerepo=* --enablerepo=local*
28. sudo yum list tcpdump
29. tcpdump -i enp5s0
30. tcpdump -nn -i enp5s0
31. sudo tcpdump -nn -c5 -i enp5s0 '!port 22'  
---  
# DAY 3  
---  
__Google Stenographer__

- Fast write speed, slow read speed.
- Written in GO
1. sudo yum list stenographer 
2. sudo yum install stenographer / hit y
3. cd /etc/stenographer
4. ls -l
5. sudo vi config
6. In the "Packets Directory" write "/data/stenographer/packets"
    In the "Index Directory" "/data/stenographer/index"
    interface = "enp5s0"
    stenotypepath = "/bin/stenotype"
7. :wq!
8. sudo mkdir /data/stenographer/packets 
9. sudo mkdir /data/stenographer/index
10. cd /data/stenographer
11. pwd
12. ls -l
13. sudo chown -R stenographer:stenographer /data/stenographer
14. sudo stenokeys.sh stenographer stenographer
15. cd /etc/stenographer/cert
16. sudo systemctl start stenographer 
17. sudo systemctl status stenographer 
18. sudo systemctl enable stenographer 
19. ^enable^status
20. __sudo journalctl -xeu stenographer__
21. ping 192.168.2.20
22. sudo stenoread 'host 192.168.2.20' -nn 'src host 192.168.2.20' 
23. cd /data/stenographer
24. watch ls -al  
---  
# Suricata  
---  
- sudo yum install suricata / y
- sudo -s 
- cd /etc/suricata
- vi suricata.yaml
    - :set nu
    - Line 56: /data/suricata 
    - Line 60: enabled: no 
    - Line 76: enabled: no
    - Line 404: enabled: no
    - Line 557: enabled: no 
    - Line 580: interface: enp5s0
    - Line 582: uncomment threads: auto
- :wq
- cd /etc/sysconfig/
- sudo vi /etc/sysconfig/suricata  
    - OPTIONS="--af-packet=enp5s0 --user suricata "
- :wq
- sudo suricata-update add-source local-emerging-threats https://192.168.2.20:8080/emerging.rules.tar
- sudo suricata-update 
- cd /data/
- sudo chown -R suricata: /data/suricata
- ls -l
- sudo systemctl start suricata
- ^start^enable
- ^enable^status
- curl -LO 192.168.2.20:8080/all-class-files.zip  
- cd /data/suricata
- cat eve.json | jq