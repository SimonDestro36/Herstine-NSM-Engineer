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
32. /var/log/audit, /var/temp
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