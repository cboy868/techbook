一、自动安装脚本(全部国内地址)

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

二、也可以手动安装

1.通过命令删除之前的brew、创建一个新的Homebrew文件夹
```
sudo rm -rf /usr/local/Homebrew
sudo mkdir /usr/local/Homebrew
```

2.git克隆（速度还是不好看文章尾部的扩展说明-1）
```
sudo git clone https://mirrors.ustc.edu.cn/brew.git /usr/local/Homebrew
```

3.删除原有的brew，创建一个新的
```
sudo rm -f /usr/local/bin/brew
sudo ln -s /usr/local/Homebrew/bin/brew /usr/local/bin/brew
```

4.创建core文件夹、克隆
```
sudo mkdir -p /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
sudo git clone https://mirrors.ustc.edu.cn/homebrew-core.git /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
```
（下面两句非必须操作）如果需要brew-cask的话，运行：

```
sudo mkdir -p /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask
sudo git clone https://mirrors.ustc.edu.cn/homebrew-cask.git /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask
```

5.删除之前brew环境，重新创建：
```
sudo rm -rf /usr/local/var/homebrew/ 
sudo mkdir -p /usr/local/var/homebrew
sudo chown -R $(whoami) /usr/local/var/homebrew
```

6.最后一步,获取权限 运行更新（三句话分开运行）
```
sudo chown -R $(whoami) /usr/local/Homebrew
HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
brew update
```
7.最后设置：设置环境变量，再运行下面两句后，重启终端：

```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
```

## 扩展说明，可有可无
1.第二步的https://mirrors.ustc.edu.cn/brew.git可以替换为下面任意一个：
```
https://mirrors.aliyun.com/homebrew/brew.git
https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
```

2.第四步的https://mirrors.ustc.edu.cn/homebrew-core.git可以替换为下面任意一个：
```
https://mirrors.aliyun.com/homebrew/homebrew-core.git
https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
```
如果更换了源依旧速度慢，换下稳定网络，例如手机4G热点或者用网线。

如果有问题自检试试：brew doctor


## 报错处理
安装完报如下错误
```
wansq$ brew doctor
-e:1:in `<main>': undefined method `canonical_segments' for #<Gem::Version "2.3.7"> (NoMethodError)
==> Downloading https://mirrors.ustc.edu.cn/homebrew-bottles/bottles-portable-ruby/portable-ruby-2.6.3.mavericks.bottle.tar.gz
Already downloaded: /Users/wansq/Library/Caches/Homebrew/portable-ruby-2.6.3.mavericks.bottle.tar.gz
Error: Checksum mismatch.
Expected: ab81211a2052ccaa6d050741c433b728d0641523d8742eef23a5b450811e5104
  Actual: c040a7a94808917bb5f9ddc53de59c927785ea8b9473431d52d0df0164982008
 Archive: /Users/wansq/Library/Caches/Homebrew/portable-ruby-2.6.3.mavericks.bottle.tar.gz
To retry an incomplete download, remove the file above.
Error: Failed to install vendor Ruby.
```
处理方式
```
rm /Users/wansq/Library/Caches/Homebrew/portable-ruby-2.6.3.mavericks.bottle.tar.gz

brew doctor
```