### 1，Gdb 调试 报错，没有权限

报错：

~~~shell
warning: Error disabling address space randomization: Operation not permitted
 Cannot create process: Operation not permitted
 During startup program exited with code 127
~~~

解决：

在创建容器的时候加上参数

~~~shell
docker run --privileged -it docker-kali-shared /bin/bash
~~~

### 2，不能连接容器workbench failed

报错信息如下：

VSCODE 远程 _workbench.downloadResource failed

原因是：服务器上不能下载vscode_server 文件

解决方式：

在root 跟目录下会有 或是直接的 cd ~ 就会看到 .vscode-server文件夹

**Step 1：获得 commit-id**

可以看到里面有个很长数字串的文件夹，如下：

进入到6cba118ac49a1b88332f312a8f67186f7f3c1643 文件夹里面，会看到其中的压缩文件vscode-server-linux-x64.tar.gz就是需要手动下载，并长传到这里的。原先是因为不能下载文件大小为0.

**Step2：压缩文件获取方式：**

然后再浏览器输入

~~~
https://update.code.visualstudio.com/commit:$COMMIT_ID/server-linux-x64/stable  
~~~

注意COMMIT_ID要替换成复制下来的字符串（$也要替换别忘了。。。。）比如我的就是

~~~
https://update.code.visualstudio.com/commit:6cba118ac49a1b88332f312a8f67186f7f3c1643/server-linux-x64/stable
~~~

（后面的stable 是vscode 的版本。可以在vscode 连接的错误信息中看到是那个）

**Step3：解压文件**

将文件放到目录：/root/.vscode-server/bin/6cba118ac49a1b88332f312a8f67186f7f3c1643 下

把其他的文件都删除，然后运行：tar -xvf vscode-server-linux-x64.tar.gz --strip-components 1

重新启动ssh ： service ssh restart 就可以正常使用了。

词方法也适用于连接服务器和服务器内容器。

**Step4：验证方式**

在服务器上使用 

~~~shell
ssh –p 51021 [root@127.0.0.1](mailto:root@127.0.0.1) 
~~~

可能出现，权限禁止。需要在 vim /etc/ssh/sshd_config 

修改 PermitRootLogin yes #允许root用户使用ssh登录

重新的启动ssh 即可。

**Know_host 问题：**

在进行本地验证的时候能回出现，以下错误：

大体意思就是，秘钥不匹配：

解决方式很简单，删除 ~/.ssh/know_host 即可。

ssh会把你每个你访问过计算机的公钥(public key)都记录在~/.ssh/known_hosts。当下次访问相同计算机时，OpenSSH会核对公钥。如果公钥不同，OpenSSH会发出警告，避免你受到DNS Hijack之类的。

### 3, 比较不同工程的文件

~~~shell
https://blog.csdn.net/weixin_45277161/article/details/131479467
~~~

### 4，折叠显示打开文件

step 1：ctrl + shift + p 

step 2：点击进入： Preference

step 3：再输入：workbench.editor.wrapTabs

