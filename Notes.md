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

