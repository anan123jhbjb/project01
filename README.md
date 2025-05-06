# project01  漏洞复现                                                                             



start.......安装配置metarget靶场
1.拉取代码
root@VM-8-3-ubuntu:~# git clone https://github.com/Metarget/metarget.git

2.切换目录
root@VM-8-3-ubuntu:~# cd metarget/

3.安装依赖
root@VM-8-3-ubuntu:~/metarget# pip3 install -r requirements.txt

![image](https://github.com/user-attachments/assets/9ee7780c-38a3-484c-8455-34d2c8c8e46b)







end


start.......TerraformGoat安装部署
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




