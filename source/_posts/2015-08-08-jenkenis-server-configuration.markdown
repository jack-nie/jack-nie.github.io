---
layout: post
title:  " Jenkins CI服务器搭建"
date:   2015-08-08
keywords: ["Server"]
description: "Jenkins CI服务器搭建"
category: "Server"
tags: ["Server"]
---

### Java环境配置

首先检查服务器上安装的Java的版本,如果低于1.7,需要将旧版本的Java移除,然后安装新的版本.

     Ruby Code:
     sudo yum remove java
     sudo yum install java-1.7.0-openjdk

然后执行 java -version,如果你看到如下输出,则说明Java环境已经安装好了.

     Ruby Code:
     java version "1.7.0_79"
     OpenJDK Runtime Environment (rhel-2.5.5.1.el6_6-x86_64 u79-b14)
     OpenJDK 64-Bit Server VM (build 24.79-b02, mixed mode)

### Jenkins 安装

    Ruby Code:
    sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
    sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key (http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key)
    sudo yum install jenkins

启动Jenkins服务

    Ruby Code:
    chkconfig jenkins on
    sudo service jenkins start

这个时候可以在服务器上通过执行一下命令,验证Jenkins服务器是否安装好了.

    curl -Xget http://localhost:8080

### NodeJs安装

Rails Asset Pipeline要求Javascript Runtime， 这里通过安装NodeJS来提供

    Ruby Code:
    wget http://nodejs.org/dist/v0.8.18/node-v0.8.18.tar.gz
    tar xf node-v0.8.18.tar.gz
    cd node-v0.8.18
    ./configure
    make
    sudo make install

### Nginx代理配置

这一步主要是为了可以通过80端口访问Jenkins.

由于服务器上面已经安装好了Nginx,所以可以不用安装Nginx,直接对其进行配置即可.

    Ruby Code:
    cd /etc/nginx/sites-available
    touch jenkins.conf

用编辑器打开刚刚创建好的jenkins.conf文件,加入如下代码.

    Ruby Code:
    upstream app_server {
        server 127.0.0.1:8080 fail_timeout=0;
    }

    server {
        listen 80;
        server_name exapmle.com;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
            proxy_set_header Host $http_host; 
            proxy_set_header Host $host:$server_port; 
            proxy_redirect off; 
            proxy_pass http://localhost:8080; 
            proxy_redirect http://example.com:8080/ http://example.com/; 
            port_in_redirect off; 
        }
    }

然后执行

    Ruby Code:
    cd /etc/nginx/sites-enabled/
    ln -s ../sites-available/jenkins.conf

加载该配置文件

    sudo service nginx reload

### 手动安装Rails环境

安装Jenkins的时候，已经自动帮我们添加了jenkins用户，其Home目录位于`/var/lib/jenkins/`。但是不能登录。先修改用户属性让jenkins用户可登录，用bash作为默认shell.

    sudo usermod -s /bin/bash jenkins

以jenkins用户登录系统，安装rvm以及ruby 2.2.2

    Ruby Code:
    sudo su - jenkins
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
    \curl -L https://get.rvm.io | bash -s stable
    rvm install 2.2.2
    gem install bundler

如果在安装的过程中出现了`curl: (23) Failed writing body (0 != 16150)`的错误,则需要运行如下命令:

    sudo chown -R jenkins:jenkins /usr/local/rvm

或者查看磁盘空间,看是否已满!


有些工程里用.rvmrc文件来指定rvm配置，比如使用哪一版本的ruby，在`/var/lib/jenkins/.rvmrc`加上以下这句，方便自动信任工程.rvmrc里的内容

    export rvm_trust_rvmrcs_flag=1

最后确保有以下类似内容`/var/lib/jenkins/.bashrc`，主要用来初始化rvm环境：

    Ruby Code:
    PATH=$PATH:/usr/local/bin:$HOME/.rvm/bin # Add RVM to PATH for scripting
    if [[ -s "$HOME/.rvm/scripts/rvm" ]]; then . "$HOME/.rvm/scripts/rvm"; fi

注意: 如果系统中已经安装了rvm和Ruby On Rails环境,则不需要再安装rvm.

### Bitbucket设置

为jenkins用户生成ssh key:

    ssh-keygen -t rsa

打开Bitbucket，找到你想运用Jenkins的repository，把刚刚生成的公钥（在`~/.ssh/id_rsa.pub`里）添加到相应repositroy的Deployment Keys。所谓Deployment Keys只有读取权限，没有push权限。
要添加Deployment Keys需要拥有该项目的管理员权限,进入项目主页,进入settings→Deployment Keys→ add keys即可看到如下表单,Label随便填,SSH key为刚刚生成的Key,填写完毕后点击Add key.
以jenkins用户运行一次以下命令，首次连接系统会问是否信任此host，回答yes将Bitbucket加入ssh信任host里。注意需要*将[username]和[repository]换成你Bitbucket上相应的用户及代码库。*

    git ls-remote -h git@bitbucket.org:[username]/[repository].git HEAD 

如果没有错误信息,则表明配置正确.

### Jenkins安全设置

Jenkins刚安装完，是允许所有访问Jenkins的人进行所有操作的，这样当然不安全，不过Jenkins提供了相当全面的权限控制，稍微设置下即可。

*  进入Manage Jenkins -> Configure Global Security，选上Enable Security，在出现的选项中，再选上Jenkins’s own user database以及下面的Allow users to sign up和Logged-in users can do anything，保存
*  返回Jenkins主页面，这时右上角会有sign up，点击进去注册一个新用户
*  注册完成后再进入设置，把那个Allow users to sign up取消掉，不再让其他人注册
*  进入这个用户的设置界面，会有API Token，点Show API Token，记下这个token，在Bitbucket的设置上会用到

### Jenkins项目设置

进行这步之前,请确保已经为Jenkins安装了git插件, 插件的安装可通过Manage Jenkins → Manage Plugins 安装.

*  点New job新建一个free style的。
*  SCM那里选Git，在Repository URL里填上Bitbucket上这个工程的地址。
*  Build Triggers那里选Trigger builds remotely，在Authentication Token填入随机字串，这个token待会在Bitbucket设置的时候会用到，记下。

### Bitbucket项目设置

*  进入工程的admin，选Services，在Service下拉框中选Jenkins，按以下填入：
    *  Module name：留空
    *  Token：这就是在Jenkins的项目里，你自己填入随机字串的那个Authentication Token
    *  Project name：Jenkins上的工程名字
    *  Endpoint：http://[Jenkins User]:[API Token]@[hostname]/，如：http://admin:xxxxxxxxxxx@example.com/

这样一来，每次你push代码到Bitbucket，它都会通知Jenkins去做你要让它帮你做的事情！
*确保Authentication Token与API Token都跟你在Jenkins上的一致。*

Execute Shell在Jenkins那build下选择execute shell，在里面填入你让Jenkins帮你做的事！以下例子，是每次push代码到Bitbucket，Jenkins都会帮你在新代码下执行rspec测试.

    Ruby Code:
    #!/bin/bash -x
    source /etc/profile.d/rvm.sh       # load rvm environment
    rvm use 2.2.0           # use ruby 2.2.0
    bundle install          
    read -d '' database_yml <<"EOF"  
    default: &default
      adapter: mysql2
      encoding: utf8
      pool: 5
      username: "username"
      password: "password"

    test:
      <<: *default
      database: p2p_test
    EOF
    echo "$database_yml" > config/database.yml
    rake db:test:prepare RAILS_ENV=test
    rake spec   
    
