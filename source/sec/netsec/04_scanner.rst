*************
网络安全扫描
*************

Tsunami
========
Goolge开源的网络扫描器，以应对大规模网络漏洞发现和扫描。

1. `Github地址 <http://>`_ 
   

2. 扫描本机
::
   
	cd /home/kali/tsunami && \
	java -cp "tsunami-main-0.0.2-SNAPSHOT-cli.jar:/home/kali/tsunami/plugins/*" \
	  -Dtsunami-config.location=/home/kali/tsunami/tsunami.yaml \
	  com.google.tsunami.main.cli.TsunamiCli \
	  --ip-v4-target=127.0.0.1 \
	  --scan-results-local-output-format=JSON \
	  --scan-results-local-output-filename=/tmp/tsunami-output.json
