





---


# 一、clone 原来的项目
```bash
$ git clone --bare git://github.com/username/project.git
```

# 二、推送到新的gitlab
```bash
$ cd project
$ git push --mirror git@example.com/username/newproject.git
```
会提示没有权限, 在gitlab中把项目的权限保护关掉就好了

# 三、本地代码更换gitlab地址
```bash
$ git remote set-url origin git@example.com/username/newproject.git
```