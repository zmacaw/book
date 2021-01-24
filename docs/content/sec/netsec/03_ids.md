---
title: 入侵检测
---

入侵检测知识和案例等。

# suricata

1. 配置文件编辑

	- 设置eve日志文件按指定周期保存（logstash：path => ["/var/log/suricata/eve-*.json"]）
		
			outputs:
			  - eve-log:
			      filename: eve-%Y-%m-%d-%H:%M.json
			      rotate-interval: minute

	
2. 启动命令
		
		# enp2s0根据实际情况替换成镜像流量口
		suricata -c /etc/suricata/suricata.yaml -i enp2s0 -D
**参考链接：**
[在Ubuntu 18.04 LTS上使用ELK和Web前端的Suricata IDS](https://www.howtoing.com/suricata-with-elk-and-web-front-ends-on-ubuntu-bionic-beaver-1804-lts)