# 0 环境

```shell
uname -m
# 输出x86_64，则AMD
# 输出armv7l 或 arm 开头的其他值，则ARM
```

# 1 安装

```sh
# Ubuntu/Debian AMD64
curl -LO https://download.influxdata.com/influxdb/releases/influxdb2_2.7.6-1_amd64.deb
sudo dpkg -i influxdb2_2.7.6-1_amd64.deb
```

```shell
# Ubuntu/Debian ARM64
curl -LO https://download.influxdata.com/influxdb/releases/influxdb2_2.7.6-1_arm64.deb
sudo dpkg -i influxdb2_2.7.6-1_arm64.deb
```

# 2 启动服务

```shell
sudo service influxdb start
```

# 3 检查服务

```shell
sudo service influxdb status
```

如果成功，输出为以下：

```text
● influxdb.service - InfluxDB is an open-source, distributed, time series database
   Loaded: loaded (/lib/systemd/system/influxdb.service; enabled; vendor preset: enable>
   Active: active (running)
```