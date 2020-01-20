###一、监控系统组件

----------
<table>
<tbody><tr>
<td>组件名</td>  <td>功能</td> <td>端口</td>
</tr>
<tr>
<td>node_exporter</td>  <td>系统数据采集服务</td><td>9100</td>
</tr>
<tr>
<td>bigdata-exporter</td>  <td>大数据组件数据采集服务</td><td>1234</td>
</tr>
<tr>
<td>prometheus</td>  <td>监控数据拉取和存储</td>  <td>9090</td>
</tr>
<tr>
<td>grafana</td>  <td>图表展示</td> <td>3000</td>
</tr>
<tr>
<td>altermanager</td>  <td>告警程序</td> <td>9093</td>
</tr>
</tbody></table>
<br>

###二、运行环境
<br>
 - cent os 7+<br>
 - JDK1.8(8~10都可以)<br>
 - 时间同步<br>
 - 机器间免密<br>

###三、有关项目启动和配置的说明
1.启动exporter（包括node_exporter和bigdata-exporter）。 <br>
2.启动alertmanager。<br>
3.启动prometheus。<br>
4.启动grafana。<br>
`		
###四、安装说明
## node_exporter ##
1.monitor包解压后，将其中的node_exporter-0.18.1.linux-amd64分发到**所有**需要采集系统信息的机器上，目录假定为
/ddhome/monitor/node_exporter-0.18.1.linux-amd64].<br>
2.编辑文件/usr/lib/systemd/system/node_exporter.service,具体指令如下<br>

    cat >/usr/lib/systemd/system/node_exporter.service <<EOF
    [Unit]
    Description=node_exporter
    Documentation=https://prometheus.io/
    After=network.target
    
    [Service]
    Type=simple
    ExecStart=/ddhome/monitor/node_exporter-0.18.1.linux-amd64/node_exporter
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    EOF


    
3.启动node_exproter，启动命令如下<br>

    systemctl enable node_exporter
    systemctl start node_exporter
## bigdate-exporter ##
1.将java包bigdata-exporter-1.0-SNAPSHOT-jar-with-dependencies.jar上传至包含(hadoop/hbase/hive)配置信息的机器，
例如应用环境机器为192.168.55.202,目录为/ddhome/monitor/bigdata-exporter-1.0-SNAPSHOT-jar-with-dependencies.jar .<br>
2.编辑文件/usr/lib/systemd/system/bigdata-exporter.service,具体指令如下<br>

    cat >/usr/lib/systemd/system/bigdata-exporter.service <<EOF
    [Unit]
    Description=bigdata-exporter
    After=network.target
    
    [Service]
    Type=simple
    ExecStart=/usr/java/jdk1.8.0_171/bin/java -Xms500m -Xmx1000m -cp /ddhome/monitor/bigdata-exporter-1.0-SNAPSHOT-jar-with-dependencies.jar BigdataExporter
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    EOF
3.启动bigdata-exporter，启动命令如下<br>

	systemctl enable bigdata-exporter
	systemctl start bigdata-exporter
## prometheus ##
1.monitor包解压后，将prometheus-2.15.1.linux-amd64分发到集群中任意一台机器，目录假定为<br>
/ddhome/monitor/prometheus-2.15.1.linux-amd64<br>
2.配置prometheus.yml文件，主要是需要采集的服务地址，内容如下<br>

    global:
      scrape_interval: 60s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
      evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
      scrape_timeout: 10s
    
    alerting:
      alertmanagers:
      - static_configs:
    	- targets:
      	  - localhost:9093
    
    rule_files:
       - "/ddhome/monitor/prometheus-2.15.1.linux-amd64/rules/rule.yml"	#alertmanager rule config
    
    scrape_configs:
      - job_name: 'linux' 
    	static_configs:
    	- targets: ['localhost:9100','192.168.55.202:9100','192.168.55.19:9100','192.168.55.82:9100','192.168.55.61:9100','192.168.55.209:9101','192.168.55.232:9101','192.168.55.113:9100']
 
      - job_name: 'bigdata'
    	static_configs:
    	- targets: ['192.168.55.202:1234']
3.编辑文件/usr/lib/systemd/system/prometheus.service，具体指令如下<br>

    [Unit]
    Description=Prometheus
    Documentation=https://prometheus.io/
    After=network.target
    
    [Service]
    Type=simple
    ExecStart=/ddhome/monitor/prometheus-2.15.1.linux-amd64/prometheus --config.file=/ddhome/monitor/prometheus-2.15.1.linux-amd64/prometheus.yml
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    EOF
4.启动prometheus,启动命令如下<br>

    systemctl enable prometheus
    systemctl start prometheus
## grafana ##
1.monitor包解压后，将prometheus-2.15.1.linux-amd64分发到**prometheus所在机器**，目录假定为<br>
/ddhome/monitor/grafana-6.5.2<br>
2.编辑文件/usr/lib/systemd/system/grafana-server.service，具体指令如下<br>

    cat >/usr/lib/systemd/system/grafana-server.service <<EOF
    [Unit]
    Description=Grafana
    After=network.target
    
    [Service]
    Type=notify
    ExecStart=/ddhome/monitor/grafana-6.5.2/bin/grafana-server -homepath /ddhome/monitor/grafana-6.5.2
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    EOF
3.启动grafana，启动命令如下<br>

    systemctl enable grafana-server
    systemctl start grafana-server
## alertmanager ##
1.monitor包解压后，将alertmanager-0.20.0.linux-amd64分发到**prometheus所在机器**，目录假定为<br>
/ddhome/monitor/alertmanager-0.20.0.linux-amd64<br>
2.编辑文件/usr/lib/systemd/system/alertmanager.service，具体指令如下<br>

    cat >/usr/lib/systemd/system/alertmanager.service <<EOF
    [Unit]
    Description=alertmanager
    Documentation=https://prometheus.io/
    After=network.target
    
    [Service]
    Type=simple
    ExecStart=/ddhome/monitor/alertmanager-0.20.0.linux-amd64/alertmanager --config.file=/ddhome/monitor/alertmanager-0.20.0.linux-amd64/alertmanager.yml
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    EOF
3.启动alertmanager，启动命令如下<br>

    systemctl enable alertmanager
    systemctl start alertmanager
