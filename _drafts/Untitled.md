2、通过sudo su命令切换到root用户

sudo su //切换到root用户

3、修改SSH配置文件/etc/ssh/sshd_config

vi /etc/ssh/sshd_config //编辑文件

找到PermitRootLogin和PasswordAuthentication

# Authentication:LoginGraceTime 120PermitRootLogin yes //默认为no，需要开启root用户访问改为yesStrictModes yes # Change to no to disable tunnelled clear text passwordsPasswordAuthentication yes //默认为no，改为yes开启密码登陆

4、重启SSH服务使修改生效

/etc/init.d/ssh restart
