# Prometheus

## Server端安装 prometheus / alertmanager / node_expoter
### 一、安装prometheus server
##### 0.获取软件包(略)、解压复制二进制文件
```
tar -xf ./packages/alertmanager-0.15.3.linux-amd64.tar.gz
mv alertmanager-0.15.3.linux-amd64/alertmanager /usr/local/sbin/
```
##### 1.创建运行用户

```
adduser -M -s /sbin/nologin prometheus
```
##### 2.创建用户所有者目录

```
# 创建prometheus server运行的目录主要存放配置文件
mdkir /usr/local/share/prometheus/prometheus_server -p
chown -R prometheus:prometheus  /usr/local/share/prometheus
# 创建prometheus server数据存储目录
mkdir /var/lib/prometheus/data
chown -R prometheus:prometheus  /var/lib/prometheus/data


# 创建prometheus alertmanager运行的目录主要存放配置文件
mdkir /usr/local/share/prometheus/prometheus_alertmanager
# 创建prometheus alertmanager数据存储目录
mkdir /var/lib/alertmanager/data -p
chown -R prometheus:prometheus  /var/lib/alertmanager/data
```
##### 3.创建prometheus server systemd文件

```
cp ./conf/systemd_conf/prometheus.service /etc/systemd/system/

# 服务操作
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
systemctl status prometheus

# 保证端口以及进程
ps aux | grep prometheus | grep -v grep  && ss -tunl | grep 9090 | grep -v grep
```
##### 4.访问:

```
http://xxxxxxx:9090
```
### 二、安装prometheus AlertManager
##### 0.获取软件包(略)、解压复制二进制文件
```
tar -xf ./packages/prometheus-2.4.2.linux-amd64.tar.gz
mv prometheus-2.4.2.linux-amd64/prometheus /usr/local/sbin/
```
##### 1.创建prometheus alertmanager systemd文件
```
cp ./conf/systemd_conf/alertmanager.service /etc/systemd/system/

# 服务操作
systemctl daemon-reload
systemctl start alertmanager
systemctl enable alertmanager
systemctl status alertmanager
# 保证端口以及进程
ps aux | grep alertmanager | grep -v grep  && ss -tunl | grep 9093 | grep -v grep
```
##### 2.将alertmanager跟prometheus server结合
修改prometheus server配置:

```
vim /usr/local/share/prometheus/prometheus_server/prometheus.yml
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']
```
##### 3.访问:
```
http://xxxxxxx:9093
```

### 三、安装Node Exporter收集主机信息
数据收集的任务由不同的 exporter 来完成，如果要收集 linux 主机的信息，可以使用 node exporter。然后由 Prometheus Server 从 node exporter 上拉取信息。
##### 0.获取软件包(略)、解压复制二进制文件
```
tar -xf ./packages/node_exporter-0.17.0-rc.0.linux-amd64.tar.gz
mv node_exporter-0.17.0-rc.0.linux-amd64/node_exporter /usr/local/sbin/
```
##### 1.把 node exporter 也配置成通过 systemd 管理, 创建文件 /etc/systemd/system/node-exporter.service
```
cp ./conf/systemd_conf/node_exporter.service  /etc/systemd/system/node-exporter.service


#服务操作
systemctl daemon-reload
systemctl enable node-exporter
systemctl start node-exporter
systemctl status node-exporter

# 保证端口以及进程
ps aux | grep node_exporter | grep -v grep  && ss -tunl | grep 9100 | grep -v grep
```
Prometheus Server 可以从不同的 exporter 上拉取数据，对于上面的 node exporter 我们可以利用 Prometheus 的static_configs 来拉取 node exporter 的数据。`conf/prometheus_server_conf/prometheus/prometheus.yml`中已经定义好了

```
  - job_name: 'node'
    static_configs:
    - targets: ['66.112.211.12:9100']
```

重启各服务；重启后 prometheus 服务会每隔 15s 从 node exporter 上拉取一次数据。
Prometheus Server 提供了简易的 WebUI 可以进数据查询并展示，它默认监听的端口为 9090。接下来我们进行一次简单的查询来验证本文安装配置的系统。


关于各项指标的规则还需要通过编写rule条目来实现；这里简单实现了wechat跟email的报警配置，具体可看规则配置文件`conf/prometheus_server_conf/prometheus/rules/hoststas-alert.rules`以及报警触发配置文件`conf/alertmanager_conf/alertmanager.yml`；

#### 报警效果图:
###### 微信:
![-300](https://github.com/guomaoqiu/prometheus_notes/blob/master/screenshots/wechat_alert.png)
###### 邮件:
![-w300](https://github.com/guomaoqiu/prometheus_nodes/blob/master/screenshots/email_alert.png)
### 四、可视化
##### 0.安装grafana(docker)
```
docker run -d -p 3000:3000 grafana/grafana
```
##### 1.访问配置导入node_exporter dashaboard模板
```
# 模板位置: conf/grafana_conf/node_exporter_monitor_templates.json
```

###### 2. 数据可视化效果图:
![](https://github.com/guomaoqiu/prometheus_nodes/blob/master/screenshots/monitor_dashboard.png)


