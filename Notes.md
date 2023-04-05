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
- sudo suricata-update add-source local-emerging-threats http://192.168.2.20:8080/emerging.rules.tar
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
---  
# DAY 4  
---  
__ZEEK__  

---

- /usr/bin - binaries
- /etc/zeek - configs
- /var/log - data
- /usr/share/zeek  

1. sudo yum install zeek zeek-plugin-kafka zeek-plugin-af_packet
2. cd /etc/zeek
3. sudo vi /etc/zeek/networks.cfg  -- no changes needed
4. sudo vi /etc/zeek/zeekctl.cfg
    - :set nu
    - Line 67: /data/zeek
    - Bottom line add: lb_custom.InterfacePrefix=af_packet::
5. :wq
6. sudo vi /etc/zeek/node.cfg  
    - :set nu
    - Lines 8-11: Comment out -not doing a standalone
    - Lines 16-18: Uncomment -for load balancing
    - Line 23: add in pin_cpus=1
    - Lines 25-32: uncomment 
    - Line 32: Interface=enp5s0
    - Add Line at 33: lb_method=custom
    - Add Line at 34: lb_procs=2 
    - Add Line at 35: pin_cpus=2, 3
    - Add line at 36: env_vars=fanout_id=77
7. :wq
8. sudo mkdir /usr/share/zeek/site/scripts
9. cd /usr/share/zeek/site/scripts
10. pwd
11. sudo curl -LO http://192.168.2.20:8080/zeek_scripts/afpacket.zeek
12. sudo vi /usr/share/zeek/site/scripts/afpacket.zeek
13. sudo curl -LO http://192.168.2.20:8080/zeek_scripts/extension.zeek
14. sudo vi /extension.zeek
15. sudo vi /usr/share/zeek/site/local.zeek
    - Shift G
    - Add Line at bottom: @load ./scritpts/afpacket.zeek
    - Add Line: @load ./scripts/extension.zeek
16. :wq
17. cd /data
18. ls -l
19. sudo chown -R zeek: /etc/zeek
20. sudo chown -R zeek: /data/zeek
21. sudo chown -R zeek: /usr/share/zeek
22. sudo chown -R zeek: /usr/bin/capstats
23. sudo chown -R zeek: /var/spool/zeek
24. sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/zeek
25. sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/capstats
26. sudo getcap /usr/bin/zeek
27. sudo getcap /usr/bin/capstats
28. sudo vi /etc/systemd/system/zeek.service
    - [Unit]
    - Description=Zeek Network Analysis Engine
    - After=network.target

    - [Service]
    - Type=forking
    - User=zeek
    - ExecStart=/usr/bin/zeekctl deploy
    - ExecStop=/usr/bin/zeekctl stop

    - [Install]
    - WantedBy=multi-user.target
29. :wq
30. sudo cd /usr/share/zeek/site/scripts
31. sudo vi /usr/share/zeek/site/local.zeek
32. sudo systmctl deamon-reload
33. sudo systemctl start zeek
34. ls -l /data/zeek/current/
35. sudo systemctl enable zeek


---
__KAFKA__

---

1. sudo yum install kafka zookeeper 
2. sudo vi /etc/zookeeper/zoo.cfg
    - No changes
3. sudo systemctl start zookeeper
4. ^start^status
5. ^status^enable
6. sudo vi /etc/kafka/server.properties
    - :set nu
    - Line 31: uncomment and add PLAINTEXT://172.16.50.100:9092
    - Line 36: uncomment and add PLAINTEXT://172.16.50.100:9092
    - Line 60: /data/kafka
    - Line 65: num.partitions=3
    - Line 107: log.retention.bytes=90000000000
    - Line 123: zookeeper.connect=127.0.0.1:2181
7. :wq
8. sudo chown -R kafka: /data/kafka
9. sudo firewall-cmd --add-port=9092/tcp --permanent
10. sudo firewall-cmd --add-port=2181/tcp --permanent
11. sudo firewall-cmd --reload
12. sudo systemctl start kafka
12. ^start^status
13. ^status^enable
14. sudo cd /usr/share/zeek/site/scripts
15. pwd
16. curl -LO http://192.168.2.20:8080/zeek_scripts/kafka.zeek
17. sudo vi kafka.zeek
    - Line 7: "172.16.50.100:9092"
18. :wq
19. cd ..
20. sudo vi local.zeek
21. shift G
22. at the bottom add @load ./scripts/kafka.zeek
23. sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.50.100:9092 --list
24. sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.50.100:9092 --topic zeek-raw 
---  
__FSF__  

---

1. sudo yum install fsf
2. sudo vi /opt/fsf/fsf-server/conf/config.py
    - Line 9: LOG_PATH : '/data/fsf/logs',
    - Line 10: YARA_PATH : '/var/lib/yara-rules/rules.yara',
    - Line 11: PID_PATH : '/run/fsf/fsf.pid',
    - Line 12: EXPORT_PATH : '/data/fsf/archive',
    - Line 18: IP_ADDRESS : "172.16.50.100",
3. :wq
4. cd /data
5. cd /fsf
6. sudo mkdir -p /data/fsf/{logs,archive}
7. ll
8. sudo chown -R fsf: /data/fsf
9. sudo chmod -R 755 /data/fsf
10. cd ..
11. ll
12. sudo vi /opt/fsf/fsf-client/conf/config.py
    - Line 9: '172.16.50.100'
13. sudo vi /etc/systemd/system/fsf.service
    - [Unit]
    - Description=File Scanning Framework (FSF-Server) Service
    - After=network.target

    - [Service]
    - Type=forking
    - User=fsf
    - Group=fsf
    - WorkingDirectory=/
    - PIDFile=/run/fsf/fsf.pid
    - PermissionsStartOnly=true
    - ExecStartPre=/bin/mkdir -p /run/fsf
    - ExecStartPre-/bin/chown -R fsf:fsf /run/fsf
    - ExecStart=/opt/fsf/fsf-server/main.py start
    - ExecStop=/opt/fsf/fsf-server/main.py stop
    - ExecReload=/opt/fsf/fsf-server/main.py restart

    - [Install]
    - WantedBy=multi-user.target
14. :wq
15. sudo systemctl start fsf
16. ^start^status 
17. cd 
18. touch test
19. vi test 
20. /opt/fsf/fsf-client/fsf_client.py --full test
21. sudo mkdir /data/zeek/extracted_files
22. sudo chown -R zeek: /data/zeek/extracted_files
23. sudo chmod 0755 /data/zeek/extracted_files
24. sudo vi /usr/share/zeek/site/scripts/extract_files.zeek
    - @load /usr/share/zeek/policy/frameworks/files/extract-all-files.zeek
    - redef FileExtract::prefix = "/data/zeek/extracted_files/";
    - redef FileExtract::default_limit = 1048576000; 
25. :wq
26. sudo vi /usr/share/zeek/site/local.zeek
    - at the bottom add @load ./scripts/extract_files.zeek
27. sudo systemctl restart zeek
28. ^restart^status
29. sudo curl -LO https://192.168.2.20:8080/putty.exe
30. ls -l /data/zeek/extracted_files/
31. cd /data/zeek/extracted_files/
32. sudo file extract-
33. sudo /usr/share/zeek/site/scripts/fsf.zeek
34. cd /usr/share/zeek/site/scripts/
35. sudo curl -LO http://192.168.2.20:8080/zeek_scripts/fsf.zeek
36. sudo vi fsf.zeek 
    - Line10: local file_path = "/data/zeek/extracted_files/";
37. sudo vi /usr/share/zeek/site/local.zeek
    - @load ./scripts/fsf.zeek
38. sudo systemctl restart zeek
39. ^restart^status
40. sudo curl -LO http://192.168.2.20:8080/putty.exe
41. cat /data/fsf/logs/rockout.log | wc -l  
---  

# Day 5  

--- 
__Rebuild Day__



---
# DAY 6  
---  

__Elasticsearch__

- Node: single instance 
- Cluster: One or more nodes 
- Roles: Purpose that a node serves
- Document: Basic Unit of info that can be indexed
- Index: Logical namespace that refers to a collection of docs
- Shard: Individual piece of an index

1. ssh admin@172.16.50.100
2. sudo yum install elasticsearch / y
    - Uninstall: sudo yum remove elasticsearch
    - sudo yum install elasticsearch-7.8.1
3. sudo chown elasticsearch: /data/elasticsearch
4. cd /data/
    -verify everything should be its own owner 
5. sudo vi /etc/elasticsearch/elasticsearch.yml
    - :set nu
    - Line 17: Uncomment: cluster.name: sg50-cluster
    - Line 23: Uncomment: node.name: sg50-node
    - Line 33: Uncomment: /data/elasticsearch
    - Line 43: Uncomment
    - Line 55: Uncomment: network.host: _eno1_
    - Line 59: Uncomment: leave port at 9200
    - Line 68: create new line and add discovery.type: single.node
        - Line 68: [172.16.50.100]
        - Line 72: [172.16.50.100]
6. :wq
7. sudo mkdir -p /usr/lib/systemd/system/elasticsearch.service.d
8. sudo vi /usr/lib/systemd/system/elasticsearch.service.d/override.conf
    - [Service]
    - LimitMEMLOCK=infinity
9. :wq
10. sudo chmod 755 /usr/lib/systemd/system/elastic.service.d
11. sudo chmod 644 /usr/lib/systemd/system/elastic.service.d/override.conf
12. sudo systemctl daemon-reload
13. sudo systemctl start elasticsearch
14. ^start^status
15. cd /etc/suricata
16. sudo systemctl enable elasticsearch
17. sudo vi /etc/elasticsearch/jvm.options
    - :set nu
    - -Xms4g
18. sudo systemctl restart elasticsearch
19. ^restart^status
20. journalctl -xeu elasticsearch
21. ss -lnt
22. sudo firewall-cmd --add-port={9200,9300}/tcp --permanent
23. firewall-cmd --reload
24. sudo curl localhost:9200
25. sudo curl localhost:9200/_cat/nodes?v
26. sudo systemctl restart elasticsearch 
27. ^restart^status
28. sudo yum install kibana-7.8.1
29. sudo vi /etc/kibana/kibana.yml
    - :set nu 
    - Line 7: server.host: "172.16.50.100"
    - Line 28: Uncomment: 172.16.50.100:9200
30. :wq
31. sudo systemctl start kibana
32. ^start^status
33. ^status^enable
34. sudo firewall-cmd --add-port=5601/tcp --permanent
35. sudo firewall-cmd --reload
36. ss -lnt
37. Go to web browser type 172.16.50.100:5601 to see kibana
38. cd              -need to be in home dir
38. sudo curl -LO http://172.168.2.20:8080/ecskibana.tar.gz
39. ll
40. sudo tar -zxvf ecskibana.tar.gz
41. ./import-index-templates.sh
42. Go back to Kibana then Dev Tools
    - GET _cat/templates?v
        - ##GET ecs-zeek-*/_mapping##
        - ##GET ecs-suricata-*/_mapping##
---  
__Beats__

- single-purpose lightweight data-shippers

1. sudo yum install filebeat-7.8.1
2. sudo mv /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bk
4. cd /etc/filebeat
5. ll
6. sudo curl -LO http://192.168.2.20:8080/filebeat.yml
7. sudo vi /etc/filebeat/filebeat.yml
    - :set nu
    - Line 11: - type: log
    -            enabled: true
                 paths:
                   - /data/fsf/logs/rockout.log
                 json.keys_under_root: true
                 fields:
                   kafka_topic: fsf-raw
                 fields_under_root: true
    - Line 22: 172.16.50.100
8. :wq
9. sudo systemctl start filebeat
10. ^start^status
11. sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.50.100:9092 --list 
12. sudo rm -rf /usr/var/filebeat/registry *not needed*
13. sudo /usr/share/kafka/bin/kafka-console.sh --bootstrap-server 172.16.50.100:9092 --topic suricata-raw --from-beginning
    - can change suricata-raw to fsf-raw or zeek-raw

---  
__Logstash__  

---

1. sudo yum install logstash-7.8.1
2. cd /etc/logstash/
3. sudo curl -LO http://192.168.2.20:8080/logstash.tar
4. ll
5. sudo tar -xvf logstash.tar
6. ll
7. cd conf.d
8. sudo vi logstash-100-input-kafka-zeek.conf
    - Line 8: bootstrap_servers => "172.16.50.100:9092"
9. :wq
10. sudo vi logstash-100-input-kafka-suricata.conf
    - Line 8: "172.16.50.100:9092"
11. :wq
12. sudo vi logstash-100-input-kafka-fsf.conf
    - Line 8: "172.16.50.100:9092"
13. :wq
14. sudo vi logstash-9999-output-elasticsearch.conf
    - :set nu
    - :%s/127.0.0.1/172.16.50.100
    - Line 2: comment out
15. :wq
16. sudo systemctl start logstash
17. ^start^status
18. ^status^enable
19. cat /var/log/logstash
20. Go to kibana
21. create index pattern 
22. name: ecs-suricata-* / ecs-zeek-* / fsf-*
23. next
24. @timestamp then create index pattern 
25. go to discover 
26. check to see if fields populate  
---
# DAY 7  
---  

__Elasticsearch Cluster__
___Not Testable___  

__Logstash__  

- Server side data processing pipeline 
- Outputs final results to a variety of destinations

1. sudo systemctl stop logstash
2. cd /etc/logstash/
3. ll
4. sudo vi pipelines.yml
5. sudo mkdir my-pipeline
6. sudo chown logstash: my-pipeline/
7. cd /my-pipeline
8. sudo vi input.conf
    - input {

    }
9. :wq
10. sudo vi filter.conf
    - filter {

        mutate {

            copy => { "[messgae]" => "[event][original"] }

        }


        json {

            source => "[message]"

            target => "[zeek][http]"

            remove_field => "[message]"

        }

    }
11. :wq
12. sudo vi output.conf
    - output {

        stdout {}

    }
13. :wq  

---
# DAY 8  
---  

__Rebuild Day__

