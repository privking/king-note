# git配置ssh

**确认已经存在的ssh key** 

windows在gitbash中执行

```sh
ls -al ~/.ssh
```

**如果没有，生成新的ssh key 并添加到ssh-agent**

生成ssh-key

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
#Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]   回车存放在默认地址

#Enter passphrase (empty for no passphrase): [Type a passphrase]
#Enter same passphrase again: [Type passphrase again]
#为ssh-key设置密码 在添加到ssh-agent时需要输入
```

私钥添加到ssh-agent

1.保证ssh-agent启用 返回经称id

```sh
eval "$(ssh-agent -s)" 
```

2.把ssh key添加到ssh-agent中

```sh
ssh-add ~/.ssh/id_rsa  #  ~/.ssh/id_rsa 为默认存放地址
```

**在github中添加ssh key公钥**

```sh
~/.ssh/id_rsa.pub # 公钥地址
C:\Users\58443\.ssh # windows中对应路径
```

settings->ssh and gpg keys -> new ssh key

**测试连接**

```sh
ssh -T git@github.com
#The authenticity of host 'github.com (20.205.243.166)' can't be established.
#RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
#Are you sure you want to continue connecting (yes/no/[fingerprint])? 
# yes
#Warning: Permanently added 'github.com,20.205.243.166' (RSA) to the list of known hosts.
#Hi privking! You've successfully authenticated, but GitHub does not provide shell access.
```

**更改远程仓库url为ssh**

查看拥有的远程仓库：

```ssh
git remote -v
```

更改远程仓库的url：

```ssh
git remote set-url origin  git@github.com:privking/king-note.git
```

查看是否修改成功

```ssh
git remote -v
```

**end**

