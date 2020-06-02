#   influxdb CQ or RP 命令

## 保留策略

### 1	tfw_system

```sql
show retention policies on tfw_system   --连续查询结果
drop retention POLICY "tfw_1d" ON "tfw_system"  --删除连续查询
CREATE RETENTION POLICY "tfw_1h" on tfw_system DURATION 1h REPLICATION 1 DEFAULT;  --默认策略 1h
CREATE RETENTION POLICY "tfw_1d" on tfw_system DURATION 1d REPLICATION 1;  --非默认策略 1d 
CREATE RETENTION POLICY "tfw_90d" on tfw_system DURATION 90d REPLICATION 1;  --非默认策略 90d 
```

### 2	telegraf

```sql
CREATE RETENTION POLICY "telegraf_1h" on telegraf DURATION 1h REPLICATION 1 DEFAULT;
CREATE RETENTION POLICY "telegraf_1d" on telegraf DURATION 1d REPLICATION 1;
CREATE RETENTION POLICY "telegraf_90d" on telegraf DURATION 90d REPLICATION 1;
CREATE RETENTION POLICY "telegraf_7d" on telegraf DURATION 1w REPLICATION 1 DEFAULT; --暂定为1周默认
```

### 3	assets_info

```sql
CREATE RETENTION POLICY "assets_1d" on "assets_info" DURATION 1d REPLICATION 1 DEFAULT;
CREATE RETENTION POLICY "assets_2d" on "assets_info" DURATION 2d REPLICATION 1 DEFAULT;
CREATE RETENTION POLICY "assets_180d" on "assets_info" DURATION 180d REPLICATION 1;
```



## 连续查询

## 1	tfw_system

```sql
SHOW CONTINUOUS QUERIES  --查询连续查询策略
DROP CONTINUOUS QUERY cq_15m ON tfw_system 	--删除
SELECT * FROM tfw_system."tfw_1d".inter_30m  --查询非默认策略
```

### 1.1	inter_speed_count

```sql
CREATE CONTINUOUS QUERY inter_speed_10m ON tfw_system BEGIN SELECT max(rx_kbytes) as rx_max,mean(rx_kbytes) as rx_mean,mean(rx_kbytes)*8/10 as rx_rate,max(tx_kbytes) as tx_max,mean(tx_kbytes) as tx_mean,mean(tx_kbytes)*8/10 as tx_rate INTO tfw_system."tfw_1d".inter_speed_count FROM interface GROUP BY time(10m),inter_name END    --10m 接口速率信息                  
```

### 1.2	inter_30m

```sql
CREATE CONTINUOUS QUERY cq_30m ON tfw_system BEGIN SELECT mean(rx_kbps) as rx_kbps,mean(rx_mbps) as rx_mbps,mean(tx_kbps) as tx_kbps,mean(tx_mbps) as tx_mbps INTO tfw_system."tfw_1d".inter_30m FROM interface GROUP BY time(30m),inter_name END   --接口数据平均信息 半小时
```

### 1.3	mem_vpp_30m

```sql
CREATE CONTINUOUS QUERY cq_mem_30m ON tfw_system BEGIN SELECT mean(memory_rate) as mem_mean INTO tfw_system."tfw_1d".mem_vpp_30m FROM system_info GROUP BY time(30m) END    --半小时内存使用率

CREATE CONTINUOUS QUERY cq_mem_1d_5 ON tfw_system BEGIN SELECT mean(*) INTO tfw_system."tfw_90d".mem_vpp_5 FROM tfw_system."tfw_1h".system_info GROUP BY time(1d,5h) END 


```

### 1.4	attack_trend

```sql
CREATE CONTINUOUS QUERY attack_trend_1m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO tfw_system."tfw_1d".attack_trend FROM syslog where LOG_BL_ip_type='1' GROUP BY time(1m),LOG_BL_src_ip,LOG_BL_src_port,LOG_BL_protoc,LOG_BL_id END  --根据src_ip,src_port,protoc统计黑名单次数 1min/次
```

### 1.5	asset_trend

```sql
CREATE CONTINUOUS QUERY asset_trend_1m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO tfw_system."tfw_1d".asset_trend FROM syslog where LOG_BL_ip_type='1' GROUP BY time(1m),LOG_BL_dest_ip,LOG_BL_dest_port,LOG_BL_protoc,LOG_BL_id END  --根据dest_ip,dest_ip,protoc统计黑名单次数 1min/次
```

### 1.6	bl_city_count_stat

```sql
CREATE CONTINUOUS QUERY bl_city_1m ON "tfw_system" BEGIN SELECT sum(count) as city_sum INTO "tfw_system"."tfw_1d".bl_city_count_stat FROM bl_city GROUP BY time(1m),city_id END  --统计攻击总数/min，保留1天
```



## 2.	telegraf

### 2.1	bl_ip

```sql
CREATE CONTINUOUS QUERY bl_ip_1m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO telegraf."telegraf_1d".bl_ip FROM syslog GROUP BY time(1m),LOG_BL_src_ip,LOG_BL_id END  --根据src_ip 和 b_id 统计拦截次数
```

### 2.2	cpu_30m

```sql
CREATE CONTINUOUS QUERY cpu_30m ON telegraf BEGIN SELECT mean(usage_user) as cpu_mean INTO telegraf."telegraf_1d".cpu_30m FROM cpu where cpu='cpu-total' GROUP BY time(30m),cpu END  --半小时cpu使用率


CREATE CONTINUOUS QUERY cpu_1d_test10_sum ON telegraf BEGIN SELECT mean(*) INTO telegraf."telegraf_90d".cpu_test10_sum FROM telegraf."telegraf_1h".cpu where cpu='cpu-total' GROUP BY time(1d,10h),cpu END 
```

### 2.3	bl_protoc

```sql
CREATE CONTINUOUS QUERY bl_protoc_10m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO telegraf."telegraf_1d".bl_protoc FROM syslog where LOG_BL_protoc='6' or LOG_BL_protoc='17' GROUP BY time(10m),LOG_BL_protoc END  --10min 统计协议条数
```

### 2.4	bl_port

```sql
CREATE CONTINUOUS QUERY bl_port_10m ON telegraf BEGIN SELECT count(LOG_BL_dest_city_id) INTO telegraf."telegraf_1d".bl_port FROM syslog GROUP BY time(10m),LOG_BL_src_port END   --10min 根据端口统计条数
```

### 2.5	bl_entire(非cq)

```sql
SELECT

      LOG_BL_dest_city_id,  LOG_BL_src_city_id::field,   LOG_BL_dest_ip,   LOG_BL_id,   LOG_BL_dest_port,  LOG_BL_ip_type,  LOG_BL_protoc,  LOG_BL_src_ip,  LOG_BL_src_port::tag into   tfw_system.tfw_system_90d.entire_log 

from 


telegraf.telegraf_1h.syslog   --降采样，默认策略中的数据插入到非默认策略中



select *  from (SELECT
     count(LOG_BL_src_ip_field) 
from  syslog   where time > now()  -1d   and LOG_BL_ip_type = '1'  group by  LOG_BL_dest_city_id,  LOG_BL_src_city_id,   LOG_BL_dest_ip,   LOG_BL_id,   LOG_BL_dest_port,  LOG_BL_ip_type,  LOG_BL_protoc,  LOG_BL_src_ip,  LOG_BL_src_port)

CREATE CONTINUOUS QUERY "bl_log_1m" on telegraf BEGIN SELECT * from (SELECT count(LOG_BL_src_ip_field) from syslog) INTO "tfw_system"."tfw_1d".bl_log_test from syslog where LOG_BL_ip_type='1' group by time(1m), LOG_BL_dest_city_id,LOG_BL_src_city_id,LOG_BL_dest_ip,LOG_BL_id,LOG_BL_dest_port,LOG_BL_ip_type,LOG_BL_protoc,LOG_BL_src_ip,LOG_BL_src_port

```

## 3.	assets_info

### 3.1	assets_at_stat

```sql
CREATE CONTINUOUS QUERY "assets_sum_180d" ON "assets_info" BEGIN SELECT sum(*) INTO "assets_info"."assets_180d".assets_at_stat_sum FROM "assets_info"."assets_2d"."assets_at_stat" GROUP BY time(24h,-8h),ips_at_ip END  --根据ip统计次数 1d

CREATE CONTINUOUS QUERY "assets_sum_test10" ON "assets_info" BEGIN SELECT sum(*) INTO "assets_info"."assets_180d".assets_at_stat_test10 FROM "assets_info"."assets_2d"."assets_at_stat" GROUP BY time(24h,10h),ips_at_ip END

```































