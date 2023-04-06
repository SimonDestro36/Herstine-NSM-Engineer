# Troubleshooting  
---  

- Gather info 
- Document solution

-Network --> Inline-tap --> Sensor 

- stenographer uses 1234, /data/stenographer is where it is written, /etc/stenographer/config

- suricata no port, /etc/suricata.yaml, eve.json

- zeek no port, output is /data/zeek, /etc/zeek/networks.cfg, zeekctl.cfg, and node.cfg

- fsf port 5800, output is rockout.log, config files /opt/fsf

- filebeat no port, filebeat.yml, 

- logstash 9300, 

---

- ip a : promisc and up
- ping!!!
- ss -lnt to determine sockets and connections
- sudo firewall-cmd --list-all
- sudo journalctl -xeu <service>
- sudo tail -f /var/log/logstash/logstash-plain
- sudo systemctl status <service>  
---  
__Stenographer__

- ping 192.168.2.20
- stenoread 'host 192.168.2.20' -nn 
- watch ls -al /data/stenographer/packets

__Suricata__

- sudo curl -LO 192.168.2.20:8080/all-class-files.zip
- cat eve.json | jq

__Kafka__

- sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.50.100 --list

__FSF__

- create a test file (it is in notes above)
- cat rockout.log | jq

__Elasticsearch__ 

- sudo curl 172.16.50.100:9200

__kibana__ 

- if you can connect to port 5601

__Filebeat__

- 

__Logstash__

- in notes above 

- watch -d curl 172.16.50.100:9200/_cat/indices?v