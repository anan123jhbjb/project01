这是我的个人空间



# project01                                                                              



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






