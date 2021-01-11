



# 一、下载rmp包
[清华园镜像](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/)


# 二、安装
```bash
$ rpm -ivh gitlab-ce-13.5.5-ce.0.el7.x86_64.rpm
警告：gitlab-ce-13.5.5-ce.0.el7.x86_64.rpm: 头V4 RSA/SHA1 Signature, 密钥 ID f27eab47: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:gitlab-ce-13.5.5-ce.0.el7        ################################# [100%]
It looks like GitLab has not been configured yet; skipping the upgrade script.

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.
  


     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md
```

# 三、修改配置文件
按照rpm 安装提示的修改
```bash
$ vim /etc/gitlab/gitlab.rb
# 修改如下内容
external_url 'http://192.168.220.111:8082'
```

# 四、让修改的配置文件生效
```bash
$ gitlab-ctl reconfigure
```
耐心等候

# 五、初始化密码
```bash
[root@docker1 lib]# gitlab-rails console 
--------------------------------------------------------------------------------
 GitLab:       13.5.5 (388201ec7a4) FOSS
 GitLab Shell: 13.11.0
 PostgreSQL:   11.9
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.3)
irb(main):001:0> u=User.where(id:1).first
=> #<User id:1 @root>
irb(main):002:0> u.password='12345678'
=> "12345678"
irb(main):003:0> u.password_confirmation='12345678'
=> "12345678"
irb(main):004:0> u.save!
Enqueued ActionMailer::MailDeliveryJob (Job ID: 661524d6-0e02-4a70-96b4-9ddc6b77f2aa) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", {:args=>[#<GlobalID:0x00007fa80d729c48 @uri=#<URI::GID gid://gitlab/User/1>>]}
=> true
irb(main):005:0> exit
```

# 登录
在浏览器中输入上面 <external_url> 配置的地址, 然后直接登录

![login](../../img/gitlab/rpm/login.png)

