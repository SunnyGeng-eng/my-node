# influxdb CQ or RP 命令

## 保留策略

### 1	tfw_system

```sql
show retention policies on tfw_system   --连续查询结果
drop retention POLICY "tfw_1d" ON "tfw_system"  --删除连续查询
CREATE RETENTION POLICY "tfw_1h" on tfw_system DURATION 1h REPLICATION 1 DEFAULT;  --默认策略 1h
CREATE RETENTION POLICY "tfw_1d" on tfw_system DURATION 1d REPLICATION 1;  --非默认策略 1d 
```

### 2	telegraf

```sql
CREATE RETENTION POLICY "telegraf_1h" on telegraf DURATION 1h REPLICATION 1 DEFAULT;
CREATE RETENTION POLICY "telegraf_1d" on telegraf DURATION 1d REPLICATION 1;
CREATE RETENTION POLICY "telegraf_90d" on telegraf DURATION 90d REPLICATION 1;
```

# 连续查询

## 2.1	tfw_system

```sql
SHOW CONTINUOUS QUERIES  --查询连续查询策略
DROP CONTINUOUS QUERY cq_15m ON tfw_system 	--删除
SELECT * FROM tfw_system."tfw_1d".inter_30m  --查询非默认策略
```

#### 2.11	inter_speed_count

```sql
CREATE CONTINUOUS QUERY inter_speed_10m ON tfw_system BEGIN SELECT max(rx_kbytes) as rx_max,mean(rx_kbytes) as rx_mean,mean(rx_kbytes)*8/10 as rx_rate,max(tx_kbytes) as tx_max,mean(tx_kbytes) as tx_mean,mean(tx_kbytes)*8/10 as tx_rate INTO tfw_system."tfw_1d".inter_speed_count FROM interface GROUP BY time(10m),inter_name END    --10m 接口速率信息

```

#### 2.12	inter_30m

```sql
CREATE CONTINUOUS QUERY cq_30m ON tfw_system BEGIN SELECT mean(rx_kbps) as rx_kbps,mean(rx_mbps) as rx_mbps,mean(tx_kbps) as tx_kbps,mean(tx_mbps) as tx_mbps INTO tfw_system."tfw_1d".inter_30m FROM interface GROUP BY time(30m),inter_name END   --接口数据平均信息 半小时

```

#### 2.13	mem_vpp_30m

```sql
CREATE CONTINUOUS QUERY cq_mem_30m ON tfw_system BEGIN SELECT mean(memory_rate) as mem_mean INTO tfw_system."tfw_1d".mem_vpp_30m FROM system_info GROUP BY time(30m) END    --半小时内存使用率
```



## 2.	telegraf

#### 2.2.1	bl_ip

```sql
CREATE CONTINUOUS QUERY bl_ip_1m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO telegraf."telegraf_1d".bl_ip FROM syslog GROUP BY time(1m),LOG_BL_src_ip,LOG_BL_id END  --根据src_ip 和 b_id 统计拦截次数
```

#### 2.2.2	cpu_30m

```sql
CREATE CONTINUOUS QUERY cpu_30m ON telegraf BEGIN SELECT mean(usage_user) as cpu_mean INTO telegraf."telegraf_1d".cpu_30m FROM cpu where cpu='cpu-total' GROUP BY time(30m),cpu END  --半小时cpu使用率
```

#### 2.2.3	bl_protoc

```sql
CREATE CONTINUOUS QUERY bl_protoc_10m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO telegraf."telegraf_1d".bl_protoc FROM syslog GROUP BY time(10m),LOG_BL_protoc END  --10min 统计协议条数
```

#### 2.2.4	bl_port

```sql
CREATE CONTINUOUS QUERY bl_port_10m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO telegraf."telegraf_1d".bl_port FROM syslog GROUP BY time(10m),LOG_BL_src_port END   --10min 根据端口统计条数
```

#### 2.2.5	bl_entire

```sql
SELECT

      LOG_BL_dest_city_id,  LOG_BL_src_city_id::field,   LOG_BL_dest_ip,   LOG_BL_id,   LOG_BL_dest_port,  LOG_BL_ip_type,  LOG_BL_protoc,  LOG_BL_src_ip,  LOG_BL_src_port::tag into   tfw_system.tfw_system_90d.entire_log 

from 


telegraf.telegraf_1h.syslog 


group by
       LOG_BL_dest_city_id, LOG_BL_src_city_id, LOG_BL_dest_ip, LOG_BL_id, LOG_BL_dest_port, LOG_BL_ip_type, LOG_BL_protoc, LOG_BL_src_ip, LOG_BL_src_port
```

