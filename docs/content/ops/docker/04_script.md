---
title: 容器的脚本工具
---

# 检查容器基础镜像版本
背景：扫描发现容器镜像存在大量漏洞，经过评估，漏洞主要来源于基础镜像，因此决定替换基础镜像修复漏洞。在修复过程中，一个仓库（repository）存在多个标签（tag），需要重新扫描最新版本的标签来确认是否已经升级，由于扫描比较耗时，于是采取检查镜像信息来查看是否是基于新基础镜像打包。
思路：根据仓库查询镜像标签-->找出最新的标签-->查看最新的镜像信息-->查看镜像特征和基础镜像特征比对，存在即为已升级
参考链接：https://stackoverflow.com/questions/32605556/how-to-find-the-creation-date-of-an-image-in-a-private-docker-registry-api-v2
``` python

import requests
import threading
import json
import os

def checkimage(rep, result, sema, num):

    with sema:
        print(str(num) + "：" + str(os.getpid()) + '：Checking-->' + rep)
        image_tags = requests.get('http://flyhi.top/v2/' + rep + '/tags/list')
        latest = []
        if 'NAME_UNKNOWN' in image_tags.text:
            if 'NAME_UNKNOWN' in result.keys():
                result['NAME_UNKNOWN'].append(rep)
            else:
                result['NAME_UNKNOWN'] = [rep]
            return()
        for tag in image_tags.json()['tags']:
            url = requests.get('http://flyhi.top/v2/' + rep + '/manifests/'+tag)
            if "errors" in url.text:
                continue
            # elif "openjdk-8u265-b01" in json.loads(url.json()['history'][-13]['v1Compatibility'])['container_config']['Cmd'][0]:
            #     if '已升级' in result.keys():
            #         result['已升级'].append(rep + ':' + tag)
            #     else:
            #         result['已升级'] = [rep + ':' + tag]
            #     continue
            else:
                latest.append((tag, json.loads(url.json()['history'][0]['v1Compatibility']).get('created')))


        # sort the list based upon created timestamp stored as the second element of the tuple
        latest.sort(key=lambda x: x[1])
        # return latest image tag from tuple
        latest_image = latest[-1][0]
        latest_url = requests.get('http://flyhi.top/v2/' + rep + '/manifests/'+ latest_image)
        # print(latest_image)
        # check = json.loads(latest_url.json()['history'][-13]['v1Compatibility'])['container_config']['Cmd'][0]
        check = str(latest_url.json()['history'])
        # print(str(check))
        if "openjdk-8u265-b01" in check or "jdk1.8.0_202" in check:
            if '已升级' in result.keys():
                result['已升级'].append(rep + ':' + latest_image)
            else:
                result['已升级'] = [rep + ':' + latest_image]
        else:
            if '未升级' in result.keys():
                result['未升级'].append(rep + ':' + latest_image)
            else:
                result['未升级'] = [rep + ':' + latest_image]

if __name__ == "__main__":
    print("请输入需要检查的镜像清单：")
    reps = []
    tharr = []
    while True:
        tmp = input()
        if tmp == '':
            break
        reps.append(tmp)
    result = {}
    print('Hold on buddy……I\'m working')
    sema = threading.Semaphore(10)
    num = 1
    for rep in reps:
        th = threading.Thread(target=checkimage, args=(rep, result, sema, num))
        th.start()
        tharr.append(th)
        num += 1
        # th.join()
    # print(result)
    for th in tharr:
        th.join()
    for k, v in result.items():
        print(k + "：")
        num = 0
        for val in v:
            print(val)
            num += 1
        print(k + "：" + str(num)) 

```