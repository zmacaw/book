---
title: go基础
---

go基础知识。

# 批量URL请求和响应

```go
package main

import (
	"bufio"
	"net/http"
	"os"
	"strings"
)

func main() {

	fname := "./domain.txt"
	dname := "./uri.txt"
	wfname := "./result.txt"

	f, err := os.Open(fname)
	if err != nil {
		panic(err)
	}
	d, err := os.Open(dname)
	if err != nil {
		panic(err)
	}
	wf, err := os.Create(wfname)
	if err != nil {
		panic(err)
	}

	fscanner := bufio.NewScanner(f)
	dscanner := bufio.NewScanner(d)
	fscanner.Split(bufio.ScanLines)
	dscanner.Split(bufio.ScanLines)
	// var lines []string

	var uris []string
	result := make(map[string][]string)

	for dscanner.Scan() {
		dline := dscanner.Text()
		if strings.ContainsAny(dline, "#") || len(dline) == 0 || dline == "\r\n" {
			continue
		} else {
			if !strings.HasPrefix(dline, "/") {
				dline = "/" + dline
			}
			uris = append(uris, dline)

			// fmt.Println(dline)

		}
	}

	for fscanner.Scan() {
		// lines = append(lines, fscanner.Text())
		fline := fscanner.Text()
		if strings.ContainsAny(fline, "#") || len(fline) == 0 || fline == "\r\n" {
			continue
		} else {
			if !strings.Contains(fline, "http:") {
				fline = "http://" + fline
			}
			for _, uri := range uris {
				url := fline + uri
				// fmt.Println(url)
				resp, err := http.Get(url)
				if err != nil {
					_, ok := result[err.Error()]
					if ok {
						result[err.Error()] = append(result[err.Error()], url)
					} else {
						result[err.Error()] = []string{url}
					}

					continue
				} else {
					// fmt.Println(resp.Status)
					_, ok := result[resp.Status]
					if !ok {
						// fmt.Println("create" + resp.Status + url)
						result[resp.Status] = []string{url}
					} else {
						// fmt.Println("append" + resp.Status + url)
						result[resp.Status] = append(result[resp.Status], url)
					}
				}
				defer resp.Body.Close()
			}
			// fmt.Println(fline)

		}

	}
	// fmt.Println(result)
	for k, v := range result {
		// fmt.Println(k, v)
		wf.WriteString(k + "\r\n" + strings.Join(v, "\r\n") + "\r\n")
	}
	defer f.Close()
	defer d.Close()
	defer wf.Close()
	// resp, err := http.Get("https://www.google.com")
	// if err != nil {
	// 	fmt.Println(err)
	// 	return
	// }
	// defer resp.Body.Close()
	// fmt.Println(resp.Status)
}

```

# 批量解析域名写入文件

```go

package main

import (
	"bufio"
	"net"
	"os"
	"strings"
)

func main() {

	fname := "./domain.txt"
	wfname := "./result.txt"

	f, err := os.Open(fname)
	wf, err := os.Create(wfname)
	if err != nil {
		panic(err)
	}

	fscanner := bufio.NewScanner(f)
	fscanner.Split(bufio.ScanLines)

	// w := bufio.NewWriter(wf)
	// var lines []string
	var errdomains []string

	for fscanner.Scan() {
		// lines = append(lines, fscanner.Text())
		if strings.ContainsAny(fscanner.Text(), "#") || len(fscanner.Text()) == 0 || fscanner.Text() == "\r\n" {
			continue
		}
		ips, err := net.LookupIP(fscanner.Text())
		if err != nil {
			// wf.WriteString(err.Error() + "\r\n")
			errdomains = append(errdomains, fscanner.Text()+"\r\n")
			continue
		}
		for _, ip := range ips {
			wf.WriteString(ip.String() + " " + fscanner.Text() + "\r\n")
		}

	}
	for _, d := range errdomains {
		wf.WriteString(d)
	}
	defer f.Close()
	defer wf.Close()

}


```

# cowrie日志结合HFish展示

``` go
package main

import (
    "github.com/HFish/core/rpc/core/"
    "strings"
    "fmt"
    "time"
    "github.com/hpcloud/tail"
    "github.com/tidwall/gjson"
)

// 上报结果结构
type Result struct {
    AgentIp     string
    AgentName   string
    Type        string
    ProjectName string
    SourceIp    string
    Info        string
    Id          string // 数据库ID，更新用 0 为新插入数据
}

var rpcClient *rpc.Client
var ipAddr string

func RpcInit() {
    rpcAddr := "172.20.205.110:7879"
    c, conn, err := rpc.Dial("tcp", rpcAddr)

    if err != nil {
        rpcClient = nil
        fmt.Println("RPC", "127.0.0.1", "连接 RPC Server 失败")
        ipAddr = ""
    } else {
        rpcClient = c
        ipAddr = strings.Split(conn.LocalAddr().String(), ":")[0]
        fmt.Println("连接RPC Server 成功")
    }
}


func ReportResult(typex string, projectName string, sourceIp string, info string, id string) string {
    var reply string

    if (rpcClient != nil) {
        // projectName 只有 WEB 才需要传项目名 其他协议空即可
        // id 0 为 新插入数据，非 0 都是更新数据
        // id 非 0 的时候 sourceIp 为空
        rpcName := "SSH高交互"

        result := Result{
            ipAddr,
            rpcName,
            typex,
            projectName,
            sourceIp,
            info,
            id,
        }
        //jso := {"AgentIp": ipAddr, "AgentName": rpcName, "Type": typex, "ProjectName":projectName, "SourceIp": sourceIp, "Info":info,"Id":id}
        err := rpcClient.Call("HFishRPCService.ReportResult", result, &reply)

        if err != nil {
            fmt.Println("RPC", "127.0.0.1", "上报上钩结果失败")
            RpcInit()
        } else {
            fmt.Println("上报上钩结果成功")
        }

        return reply
    } else {
        return "0"
    }
}



func main() {
    var sessionIdMap map[string]string
    sessionIdMap = make(map[string]string)
    RpcInit()
    fileName := "./cowrie/var/log/cowrie/cowrie.json"
    config := tail.Config{
        ReOpen:    true,                                 // 重新打开
        Follow:    true,                                 // 是否跟随
        Location:  &tail.SeekInfo{Offset: 0, Whence: 2}, // 从文件的哪个地方开始读
        MustExist: false,                                // 文件不存在不报错
        Poll:      true,
    }
    tails, err := tail.TailFile(fileName, config)
    if err != nil {
        fmt.Println("tail file failed, err:", err)
        return
    }
    var (
        line *tail.Line
        ok   bool
    )
    for {
        line, ok = <-tails.Lines
        if !ok {
            fmt.Printf("tail file close reopen, filename:%s\n", tails.Filename)
            time.Sleep(time.Second)
            continue
        }
        json := line.Text
        eventid := gjson.Get(json, "eventid").String()
        if (eventid == "cowrie.session.connect"){
            session := gjson.Get(json, "session").String()
            srcIp := gjson.Get(json, `src_ip`).String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            sensor := gjson.Get(json, `sensor`).String()

            info :=  "&&连接时间：" + timestamp + "&&攻击者<" + srcIp + ">已连接&&" + "连接信息：" + message + "&&sensor：" + sensor
            id := ReportResult("SSH", "", srcIp, info, "0")
            fmt.Println(id)
            sessionIdMap [session] = id
        }else if (eventid == "cowrie.client.version"){
            session := gjson.Get(json, "session").String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            info :=  "&&时间：" + timestamp + "&&攻击客户端：" + message
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            fmt.Println(id)

        }else if (eventid == "cowrie.client.kex"){
            session := gjson.Get(json, "session").String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            hasshAlgorithms := gjson.Get(json, `hasshAlgorithms`).String()
            info :=  "&&时间：" + timestamp + "&&客户端指纹：" + message + "&&哈希算法：" + hasshAlgorithms
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            fmt.Println(id)

        }else if (eventid == "cowrie.login.failed"){
            session := gjson.Get(json, "session").String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            info :=  "&&时间：" + timestamp + "&&登陆失败：" + message
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            fmt.Println(id)
        }else if (eventid == "cowrie.login.success"){
            session := gjson.Get(json, "session").String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            info :=  "&&时间：" + timestamp + "&&登录成功：" + message
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            fmt.Println(id)
        }else if (eventid == "cowrie.client.size"){
            session := gjson.Get(json, "session").String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            info :=  "&&时间：" + timestamp + "&&攻击终端大小：" + message
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            fmt.Println(id)
        }else if (eventid == "cowrie.command.input"){
            session := gjson.Get(json, "session").String()
            input := gjson.Get(json, `input`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            info :=  "&&时间：" + timestamp + "&&执行命令：" + input
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            fmt.Println(id)
        }else if (eventid == "cowrie.command.failed"){
            session := gjson.Get(json, "session").String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            info :=  "&&时间：" + timestamp + "&&执行命令：" + message
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            fmt.Println(id)
        }else if (eventid == "cowrie.session.closed"){
            session := gjson.Get(json, "session").String()
            message := gjson.Get(json, `message`).String()
            timestamp := gjson.Get(json, `timestamp`).String()
            info :=  "&&时间：" + timestamp + "&&连接关闭：" + message
            id := ReportResult("SSH", "", "", info, sessionIdMap[session])
            delete(sessionIdMap, session)
            fmt.Println(id)
        }
    }
}
```
