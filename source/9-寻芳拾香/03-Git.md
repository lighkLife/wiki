# Git 使用记录

## 记住用户名密码
```shell
git config --global credential.helper store
```

## 设置代理
```shell
git config --local http.proxy http://127.0.0.1:7890
git config --local https.proxy http://127.0.0.1:7890
```

## ssh 方式访问 github 失败
直接配置 `.ssh/config`
```shell
Host github.com
    Hostname ssh.github.com
    Port 443
    # user
    User lighk
```

## 覆盖更新
```shell
git fetch --all
git reset --hard origin/master
```

## 舍弃追踪某个文件
```shell
# 舍弃追踪 
git update-index --assume-unchanged utils/DB.py

# 继续追踪
git update-index --no-assume-unchanged utils/DB.py
```

## 历史树别名
```shell
git config --global alias.lg "log --graph --all --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
```