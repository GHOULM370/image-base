## 一、_Docker安装gitlab服务_

(默认已安装Docker，未安装请先尝试[docker安装](http://47.106.159.132:8080/archives/docker%E5%AE%89%E8%A3%85))

### 1、gitlab镜像拉取

```shell
# gitlab-ce为稳定版本，后面不填写版本则默认pull最新latest版本
$ docker pull gitlab/gitlab-ce
```

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/15087669-5818e213d2c0bc1ee7.png#crop=0&crop=0&crop=1&crop=1&id=hcnfs&originHeight=226&originWidth=565&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 2、运行gitlab镜像

```shell
$ docker run -d  -p 8088:443 -p 8089:80 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
# -d：后台运行
# -p：将容器内部端口向外映射
# --name：命名容器名称
# -v：将容器内数据文件夹或者日志、配置等文件夹挂载到宿主机指定目录
```

运行成功后出现一串字符串
![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/15087669-5818ed22c0bc1ee7.png#crop=0&crop=0&crop=1&crop=1&id=XdpKn&originHeight=75&originWidth=467&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 3、配置(第一种方式)

按上面的方式，gitlab容器运行没问题，但在gitlab上创建项目的时候，生成项目的URL访问地址是按容器的hostname来生成的，也就是容器的id。作为gitlab服务器，我们需要一个固定的URL访问地址，于是需要配置gitlab.rb（宿主机路径：/home/gitlab/config/gitlab.rb）。

```shell
# gitlab.rb文件内容默认全是注释
$ vim /home/gitlab/config/gitlab.rb
```

```shell
# 配置http协议所使用的访问地址,不加端口号默认为80
external_url 'http://192.168.199.231:8089' # 服务器的ip:port

# 配置ssh协议所使用的访问地址和端口
gitlab_rails['gitlab_ssh_host'] = '192.168.199.231' # 服务器的ip
gitlab_rails['gitlab_shell_ssh_port'] = 222 # 此端口是run时22端口映射的222端口
:wq # 保存配置文件并退出
```

```shell
# 重启gitlab容器
$ docker restart gitlab
```

此时项目的仓库地址就变了。如果ssh端口地址不是默认的22，就会加上ssh:// 协议头
打开浏览器输入ip地址(当gitlab端口为80，浏览器url不用输入端口号，如果端口号不是80，则打开为：ip:端口号)

(第二种方式)

检查是否有安装 docker-compose

```shell
$ docker-compose version
```

没有安装就先安装docker-compose（有安装跳过这一步）

```shell
$ curl -L https://get.daocloud.io/docker/compose/releases/download/1.12.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
			# 你可以通过修改URL中的版本，可以自定义您的需要的版本。

$ chmod +x /usr/local/bin/docker-compose

$ docker-compose version # 查看版本号，测试是否安装成功
```

建一个目录

```shell
$ sudo mkdir -p /home/data/gitlab/config
```

创建配置文件

```shell
$ vim /home/data/gitlab/docker-compose.yml
```

粘贴并修改内容然后保存：

```yaml
gitlab:
image: gitlab/gitlab-ce:11.3.6-ce.0
restart: always
hostname: '192.168.1.11'
environment:
 GITLAB_OMNIBUS_CONFIG: |
     external_url 'https://192.168.1.11:8443'
     nginx['redirect_http_to_https'] = true
     letsencrypt['enable'] = false
     nginx['ssl_certificate'] = "/etc/gitlab/nginx.pem"
     nginx['ssl_certificate_key'] = "/etc/gitlab/nginx.key"
     # Add any other gitlab.rb configuration here, each on its own line
ports:
        - 8443:8443
    volumes:
        - ./data:/var/opt/gitlab
        - ./logs:/var/log/gitlab
        - ./config:/etc/gitlab
```

自建签名

```shell
$ sudo openssl req -new -x509 -days 36500 -nodes -out config/nginx.pem \
         -keyout config/nginx.key -subj "/C=US/CN=gitlab/O=gitlab.com"
```

最后在该目录下启动docker

```shell
$ sudo docker-compose up
```

### 4、创建一个项目

第一次进入要输入新的root用户密码，设置好之后确定就行

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/15087669-6b04cfddeccf17bf.png#crop=0&crop=0&crop=1&crop=1&id=IEBEt&originHeight=550&originWidth=1200&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

下面就可以新建一个项目了，点击Create a project

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/15087669-2a40551dc13c2826.png#crop=0&crop=0&crop=1&crop=1&id=woD5H&originHeight=550&originWidth=1200&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
创建完成后：
![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/15087669-5818e2133ec0bc1ee7.png#crop=0&crop=0&crop=1&crop=1&id=woWSb&originHeight=550&originWidth=1200&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 5、登录gitlab网页

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/15087669-249a984d541801a1.png#crop=0&crop=0&crop=1&crop=1&id=quBfG&originHeight=377&originWidth=478&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## 二、用户配置账户使用

### 1、注册一个账户

首先打开公司内网部署GitLab的服务器，由于是内部员工使用，所以注册时候Username和Full name最好用自己的名字，这样管理员给用户分配项目权限的时候能够一目了然。

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/6781582-ca30fee1101f4739.png#crop=0&crop=0&crop=1&crop=1&id=bj09V&originHeight=610&originWidth=1021&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 2、管理员给用户分配权限

以管理员的身份登入gitlab，点击Settings，然后选择Members

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/6781582-054073a3d940c331.png#crop=0&crop=0&crop=1&crop=1&id=duXhS&originHeight=309&originWidth=248&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

可以通过输入名字选择要分配权限的小组成员，然后分配角色，选择权限有效时间，点击Add to Project就把人员拉近到项目中。GitLab的角色有以下四种：

-  Guest：可以创建issue、发表评论，不能读写版本库 
-  Reporter：可以克隆代码，不能提交，可以赋予测试、产品经理此权限 
-  Developer：可以克隆代码、开发、提交、push，可以赋予开发人员此权限 
-  MainMaster：可以创建项目、添加tag、保护分支、添加项目成员、编辑项目，一般GitLab管理员或者CTO才有此权限 

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/6781582-b9d4cbc699a0ded6.png#crop=0&crop=0&crop=1&crop=1&id=OBlmf&originHeight=362&originWidth=1200&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 3、配置ssh使用（个人用户看自己需求是否要配置此项，方便每次提交的时候不用输入邮箱号和密码）

gitlab可以通过HTTP和SSH去做克隆和提交代码，由于HTTP需要每次提交的时候输入邮箱号和密码，所以常用电脑上配置SSH，只要配置好了以后，下次提交的时候就方便了。SSH的方式主要是通过生成一个密钥和一个公钥，这个公钥可以使用在GitHub，GItLab，内网GitLab中。
大多数 Git 服务器都会选择使用 SSH 公钥来进行授权。系统中的每个用户都必须提供一个公钥用于授权，没有的话就要生成一个。生成公钥的过程在所有操作系统上都差不多。首先你要确认一下本机是否已经有一个公钥。
SSH 公钥默认储存在账户的主目录下的 ~/.ssh 目录。进去看看：

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/6781582-7ba3cae2bd2c79ea.png#crop=0&crop=0&crop=1&crop=1&id=MdY1b&originHeight=266&originWidth=620&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

看一下有没有id_rsa和id_rsa.pub(或者是id_dsa和id_dsa.pub之类成对的文件)，有 .pub 后缀的文件就是公钥，另一个文件则是密钥。

假如没有这些文件，甚至连 .ssh 目录都没有，可以用 ssh-keygen 来创建。该程序在 Linux/Mac 系统上由 SSH 包提供，而在 Windows 上则包含在GitBash里面里：

```shell
$ ssh-keygen -t rsa -C "XXXXXX@qq.com"

Creates a new ssh key using the provided email # Generating public/private rsa key pair.

Enter file in which to save the key (/home/you/.ssh/id_rsa):
```

然后直接三次Enter键就可以了，完了之后大概是这样：

```shell
Your public key has been saved in /home/you/.ssh/id_rsa.pub.
The key fingerprint is: # 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db XXXXXX@qq.com
```

查看公钥

```shell
vim id_rsa.pub
```

登陆GitLab账号，点击用户图像，然后 Settings -> 左栏点击 SSH keys

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/6781582-7dbf927dcb6f96b0.png#crop=0&crop=0&crop=1&crop=1&id=QMBFG&originHeight=310&originWidth=284&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

复制公钥内容，粘贴进“Key”文本区域内，取名字

点击Add Key

![](https://gitee.com/GHOULM370/mapdepot/raw/master/img/6781582-050d2c55bd5202f0.png#crop=0&crop=0&crop=1&crop=1&id=dm7x9&originHeight=671&originWidth=988&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
