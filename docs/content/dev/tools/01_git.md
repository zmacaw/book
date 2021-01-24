---
title: git的日常操作
---

git工具的使用。

# Windows和Linux换行符不一致导致git push报错
结论：Git目录位于Windows上，共享给kali虚拟机，在kali上使用git管理目录，此时会引入一个问题，Windows下换行符为\r\n，Linux换行采用\n，所以Windows文件在Linux运行会报错，需要将\r删除。
搞了一个git push脚本
```shell
GIT=`which git`
REPO_DIR=./docs/
cd ${REPO_DIR}
grep -E 'boss|password = "test" -rl ${REPO_DIR}
if [[ $? -eq 0 ]]; then
	sed -i "s/boss/zm/g;s/password = "test" -rl ${REPO_DIR}`
fi

${GIT} add --all .
${GIT} commit -m "Daily Update"
${GIT} push https://github.com/bossbook.git master -f

```
某日 git 提交的时候报错：
fatal: cannot run .git/hooks/pre-push: No such file or directory
经过查询，发现是pre-push在Windows下创建，导致在Linux下系统中换行多出了\r，可以通过以下方法确认：
1. 直接在Linux下运行pre-push
   ```  
   kali@kali:/mnt/docs$ .git/hooks/pre-push 
   -bash: .git/hooks/pre-push: /bin/sh^M: bad interpreter: No such file or directory
   ```
   **Tip:** 注意`sh^M`,多了个^M（ctrl+v ctrl+m）

==解决办法==
1. 切到`.git/hooks/`目录下执行：`dos2unix pre-push `
2. 切到`.git/hooks/`目录下执行：`cat pre-commit | tr -d '\r' > pre-push`

3. 运行：
   ```
   cp /mnt/docs/.git/hooks/pre-push /tmp/pre-push
   tr -d '\r' < /tmp/pre-push > /mnt/docs/.git/hooks/pre-push
   ```
==后续==
第一次问题得到成功解决，但是第二次发现同样的问题，同样的方法不管用了WTF T_T，不管怎么替换修改文件，问题依旧，于是决定亲手开撕。
1. 删除.git目录，重新初始化，发现在Linux下.git目录无法删除（pre-push文件删除不了）
2. 转而从Windows上删除，发现可以直接删除.git目录，但是只删除pre-push文件时，不管怎么删除、重命名pre-push文件，文件都会自动新建，@_@，什么鬼……
3. 由于当前目录从Windows挂载，Windows和kali均有可能操作文件。于是采取以下步骤确认：
   - 取消文件挂载到Linux，在宿主机Windows上删除文件，发现文件还是会自动新建，由此判断该行为为Windows导致（由前面pre-push文件换行符的特征也可以直接确定文件是在Windows上创建的）。
   - 通过[Process Monitor](../../ops/windows/01_basic.md),参考ops-Windows部分。定位到操作pre-push的进程
     ```
     运行Process Monitor
     点击：Fileter --> Filter （或者ctrl+L）
     选择：PATH --> contains --> D:\Download\docs\.git\hooks
     点击：add --> OK
     即可找到目前在编辑pre-push的进程：FUCK是dlp（数据防泄漏系统）
     ```
