# easy-elk-install

# Install Global 
yum update -y && yum install wget bind-utils net-tools nmap bash-completion epel-release perl iptables-services tcpdump -y

#Install Vmware Tools
mkdir -p /mnt/cdrom && mount /dev/cdrom /mnt/cdrom  && cp /mnt/cdrom/VMwareTools-*.tar.gz /tmp/ && cd /tmp && tar -zxvf VMwareTools-*.tar.gz && cd vmware-tools-distrib && ./vmware-install.pl

# Templates WIN,METRIC,PACKET

IP=172.16.1.30;
wget -q http://home.network-warrior.com/downloads/elk/filebeat.template.json -O /tmp/filebeat.template.json &&
wget -q http://home.network-warrior.com/downloads/elk/metricbeat.template.json -O /tmp/metricbeat.template.json &&
wget -q http://home.network-warrior.com/downloads/elk/packetbeat.template.json -O /tmp/packetbeat.template.json &&
curl -XPUT "http://$IP:9200/_template/filebeat" -d@/tmp/filebeat.template.json &&
curl -XPUT "http://$IP:9200/_template/metricbeat" -d@/tmp/metricbeat.template.json &&
curl -XPUT "http://$IP:9200/_template/packetbeat" -d@/tmp/packetbeat.template.json &&
rm -f /tmp/*beat

# Install Linux Beats

ENV=PRODUCTION;
LOGIP=172.16.1.31;
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch &&
printf '[elastic-5.x]\nname=Elastic repository for 5.x packages\nbaseurl=https://artifacts.elastic.co/packages/5.x/yum\ngpgcheck=1\ngpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch\nenabled=1\nautorefresh=1\ntype=rpm-md' > /etc/yum.repos.d/elasticsearch.repo &&
yum install filebeat metricbeat packetbeat -y &&
systemctl enable filebeat metricbeat packetbeat &&
wget -q http://home.network-warrior.com/downloads/elk/filebeat.yml -O /etc/filebeat/filebeat.yml &&
wget -q http://home.network-warrior.com/downloads/elk/metricbeat.yml -O /etc/filebeat/metricbeat.yml &&
wget -q http://home.network-warrior.com/downloads/elk/packetbeat.yml -O /etc/filebeat/packetbeat.yml &&
sed -i "s/  esolutions_environment: \*/  esolutions_environment: $ENV/" /etc/filebeat/filebeat.yml &&
sed -i "s/\*:5043/$LOGIP:5043/" /etc/filebeat/filebeat.yml &&
sed -i "s/  esolutions_environment: \*/  esolutions_environment: $ENV/" /etc/metricbeat/metricbeat.yml &&
sed -i "s/\*:5043/$LOGIP:5043/" /etc/metricbeat/metricbeat.yml &&
sed -i "s/  esolutions_environment: \*/  esolutions_environment: $ENV/" /etc/packetbeat/packetbeat.yml &&
sed -i "s/\*:5043/$LOGIP:5043/" /etc/packetbeat/packetbeat.yml &&
systemctl start filebeat metricbeat packetbeat

# Install elasticsearch

ELASTICIP=172.16.1.33;
SERVERNAME=ELASTIC-PRD1;
CLUSTERNAME=monitoring;
yum install java-1.8.0-openjdk-devel rng-tools -y &&
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch &&
printf '[elastic-5.x]\nname=Elastic repository for 5.x packages\nbaseurl=https://artifacts.elastic.co/packages/5.x/yum\ngpgcheck=1\ngpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch\nenabled=1\nautorefresh=1\ntype=rpm-md' > /etc/yum.repos.d/elasticsearch.repo &&
yum install elasticsearch -y &&
systemctl enable elasticsearch &&
wget -q http://home.network-warrior.com/downloads/elk/elasticsearch.yml -O /etc/elasticsearch/elasticsearch.yml &&
sed -i "s/cluster.name: \*/cluster.name: $CLUSTERNAME/" /etc/elasticsearch/elasticsearch.yml &&
sed -i "s/node.name: \*/node.name: $SERVERNAME/" /etc/elasticsearch/elasticsearch.yml &&
sed -i "s/network.host: \*/network.host: $ELASTICIP/" /etc/elasticsearch/elasticsearch.yml &&
systemctl start elasticsearch &&
sysctl -w vm.max_map_count=262144

# Install Logstash

ELASTICIP=172.16.1.33;
yum install java-1.8.0-openjdk-devel rng-tools -y &&
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch &&
printf '[elastic-5.x]\nname=Elastic repository for 5.x packages\nbaseurl=https://artifacts.elastic.co/packages/5.x/yum\ngpgcheck=1\ngpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch\nenabled=1\nautorefresh=1\ntype=rpm-md' > /etc/yum.repos.d/elasticsearch.repo &&
yum install logstash -y &&
systemctl enable logstash &&
wget -q http://home.network-warrior.com/downloads/elk/beats.conf -O /etc/logstash/conf.d/beats.conf &&
sed -i "s/\*:9200/$ELASTICIP:9200/" /etc/logstash/conf.d/beats.conf
systemctl start logstash

# Install Kibana

ELASTICIP=172.16.1.30;
SERVERNAME=KIBANA-PRD1;
KIBANAIP=172.16.1.33;
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch &&
printf '[elastic-5.x]\nname=Elastic repository for 5.x packages\nbaseurl=https://artifacts.elastic.co/packages/5.x/yum\ngpgcheck=1\ngpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch\nenabled=1\nautorefresh=1\ntype=rpm-md' > /etc/yum.repos.d/elasticsearch.repo &&
yum install kibana -y &&
systemctl enable kibana &&
wget -q http://home.network-warrior.com/downloads/elk/kibana.yml -O /etc/logstash/conf.d/kibana.yml &&
sed -i "s/\*:9200/$ELASTICIP:9200/" /etc/logstash/conf.d/kibana.yml &&
sed -i "s/server.name: \"\*/server.name: \"$SERVERNAME/" /etc/logstash/conf.d/kibana.yml &&
sed -i "s/server.host: \"\*/server.host: \"$KIBANAIP/" /etc/logstash/conf.d/kibana.yml &&
systemctl start kibana
