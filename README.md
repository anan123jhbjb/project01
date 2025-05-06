# project01  漏洞复现                                                                             
一、start.......安装配置metarget靶场
1.拉取代码
root@VM-8-3-ubuntu:~# git clone https://github.com/Metarget/metarget.git

2.切换目录
root@VM-8-3-ubuntu:~# cd metarget/

3.安装依赖
root@VM-8-3-ubuntu:~/metarget# pip3 install -r requirements.txt

![image](https://github.com/user-attachments/assets/9ee7780c-38a3-484c-8455-34d2c8c8e46b)

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

6.CVM SSRF










