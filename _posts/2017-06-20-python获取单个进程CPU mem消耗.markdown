---
title: python获取单个进程CPU mem消耗
---

### 脚本需要重新定义方法 ，可以减少代码
### 需要修改过滤进程bug 。如PHPFPM 多个子进程占用内存大等~
``` 
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import sys
import commands
import json
import psutil

class Get_apply_port():
    '''
    系统命令执行方法
    '''
    @staticmethod
    def system_commands(name,port):
	count = 0
	if sys.argv[1] == "mem":
		data = commands.getstatusoutput('/bin/ps aux | grep -E -v "mysqld_safe|redis-cli|grep|'+sys.argv[0]+'|*.sh" | grep '+name+' | grep '+port+' | awk {\'print $4\'}')
		split_data = data[1].split('\n')
		for i in split_data:
			x = float(i)
			count += x
		print count
	if sys.argv[1] == "cpu":
		data = commands.getstatusoutput('/bin/ps aux | grep -E -v "mysqld_safe|redis-cli|grep|'+sys.argv[0]+'|*.sh" | grep '+name+' | grep '+port+' | awk {\'print $3\'}')
                split_data = data[1].split('\n')
                for i in split_data:
                        x = float(i)
                        count += x
                print count
    '''
    根据op文件提取应用名和对应端口号
    '''
    @staticmethod
    def get_system_process():
        content = []
        get_op = commands.getstatusoutput('/bin/sh /home/playcrab/bin/service_list | grep -w `cat /home/.op/inner_ip` | awk -F \'|\' \'{print $9\":\"$6}\'')
        split_data = get_op[1].split('\n')
        split_data = list(split_data)
        for n in range(len(split_data)):
            split_data[n] = split_data[n].replace(' ','')
        for i in split_data:
            data = i.split(":")
            content.append("{\"{#PROCESS_NAME}\":\""+data[1]+"\",\"{#PORT}\":\""+data[0]+"\"}")
        redata = str(content).replace('\'','')
        print "{\"data\":",redata,"}"
    '''
    根据查询结果查询对应应用MEM占用百分比
    '''
    @staticmethod
    def get_system_mem():
	if sys.argv[2] == "nginx" or sys.argv[2] == "haproxy" or sys.argv[2] == "phpfpm":
		sys.argv[3] = "\"\""
		if sys.argv[2] == "phpfpm":
			sys.argv[2] = "php-fpm"
			Get_apply_port.system_commands(sys.argv[2],sys.argv[3])
		else:
			Get_apply_port.system_commands(sys.argv[2],sys.argv[3])
	else:
		Get_apply_port.system_commands(sys.argv[2],sys.argv[3])
    '''
    根据查询结果查询对应应用CPU占用百分比
    '''
    @staticmethod
    def get_system_cpu():
        if sys.argv[2] == "nginx" or sys.argv[2] == "haproxy" or sys.argv[2] == "phpfpm":
                sys.argv[3] = "\"\""
                if sys.argv[2] == "phpfpm":
                        sys.argv[2] = "php-fpm"
                        Get_apply_port.system_commands(sys.argv[2],sys.argv[3])
                else:
                        Get_apply_port.system_commands(sys.argv[2],sys.argv[3])
        else:
                Get_apply_port.system_commands(sys.argv[2],sys.argv[3])
if __name__ == "__main__":
    try:
        simida = sys.argv[1]
    except:
	print(sys.argv[0]+" "+ "cpu|mem|proceess" + " ""server_name" + " ""port")
    else:
        if sys.argv[1] == "cpu" :
            Get_apply_port.get_system_cpu()
        elif sys.argv[1] == "mem":
            Get_apply_port.get_system_mem()
        elif sys.argv[1] == "process":
            Get_apply_port.get_system_process()

```