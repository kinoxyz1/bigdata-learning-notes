




---
[官方文档](https://ohmyz.sh/)


# 一、安装
官方提供了两种安装方式

[安装方式](https://ohmyz.sh/#install)
```bash
kino@raomindeMacBook-Pro ~ % sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"


Cloning Oh My Zsh...
remote: Enumerating objects: 1275, done.
remote: Counting objects: 100% (1275/1275), done.
remote: Compressing objects: 100% (1230/1230), done.
remote: Total 1275 (delta 26), reused 1165 (delta 25), pack-reused 0
Receiving objects: 100% (1275/1275), 1.07 MiB | 1.76 MiB/s, done.
Resolving deltas: 100% (26/26), done.
From https://github.com/ohmyzsh/ohmyzsh
 * [new branch]      master     -> origin/master
Branch 'master' set up to track remote branch 'master' from 'origin'.
Already on 'master'
/Users/kino

Looking for an existing zsh config...
Found ~/.zshrc. Backing up to /Users/kino/.zshrc.pre-oh-my-zsh
Using the Oh My Zsh template file and adding it to ~/.zshrc.

         __                                     __
  ____  / /_     ____ ___  __  __   ____  _____/ /_
 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \
/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / /
\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/
                        /____/                       ....is now installed!


Before you scream Oh My Zsh! look over the `.zshrc` file to select plugins, themes, and options.

• Follow us on Twitter: @ohmyzsh
• Join our Discord community: Discord server
• Get stickers, t-shirts, coffee mugs and more: Planet Argon Shop

[oh-my-zsh] Insecure completion-dependent directories detected:
drwxrwxr-x  3 kino  admin  96 11 20  2021 /usr/local/share/zsh
drwxrwxr-x  2 kino  admin  64 11 20  2021 /usr/local/share/zsh/site-functions

[oh-my-zsh] For safety, we will not load completions from these directories until
[oh-my-zsh] you fix their permissions and ownership and restart zsh.
[oh-my-zsh] See the above list for directories with group or other writability.

[oh-my-zsh] To fix your permissions you can do so by disabling
[oh-my-zsh] the write permission of "group" and "others" and making sure that the
[oh-my-zsh] owner of these directories is either root or your current user.
[oh-my-zsh] The following command may help:
[oh-my-zsh]     compaudit | xargs chmod g-w,o-w

[oh-my-zsh] If the above didn't help or you want to skip the verification of
[oh-my-zsh] insecure directories you can set the variable ZSH_DISABLE_COMPFIX to
[oh-my-zsh] "true" before oh-my-zsh is sourced in your zshrc file.

➜  ~
```

# 二、修改默认的 shell

安装完成之后，在 `/bin` 目录下会多出一个 `zsh` 的文件。

```bash
$ cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

其次，macOS 在 Catalina 版本之前都是使用 `dash` 作为终端，

如果你想修改为 `zsh` ，可以使用以下命令：

```text
chsh -s /bin/zsh
```

当然，你后悔了，想改回原来的 `dash` ，同样使用上面的 `chsh` 命令就可以。

```text
chsh -s /bin/bash
```

# 三、修改主题

该装的软件都装完了，现在主要就是选择自己喜欢的风格了。

那么有哪些主题风格可以选呢？

可以通过下面的 Github 地址来查看。

```text
Github 地址：https://github.com/ohmyzsh/ohmyzsh/wiki/themes
```

里面的主题非常多，各种各样的风格都有，看你个人的喜好了。

选好了主题，下一步就是配置主题了，怎么配置呢？

此刻你可以在 iterm2 中输入以下命令

```text
vim ~/.zshrc
```

找到 `ZSH_THEME` 字段，可以看到 oh-my-zsh 的默认主题是 robbyrussell ，如果要做修改，具体操作如下：

要在 Vim 里修改文件，要先按 `i` 进入编辑模式，把 `ZSH_THEME`的值修改为你喜欢的那个主题，修改完成之后按 `esc` （电脑最左上）退出编辑模式，最后 `shift+zz` 保存并退出。

当然，你不太熟悉上面的操作，可以直接打开 `.zshrc` 的文件，然后用普通的编辑器直接修改那个 `ZSH_THEME` 的值，最后保存就好。

上面介绍的都是 oh-my-zsh默认自带了一些默认主题，存放在 `~/.oh-my-zsh/themes` 目录中。

你可以在终端输入 `cd ~/.oh-my-zsh/themes && ls` 就可以观察到。

除了这些自带的主题，还有很多很酷，很炫的定制主题。

比如，powerlevel9k 。

powerlevel9k 真的是一个很酷的东西。

那么你想用这些主题要怎么操作呢？

也很简单，比用自带的主题多了一步操作而已。

就是先把主题给下载下来。

用 powerlevel9k 为例，通过 `git clone` 下载到 oh-my-zsh 放置第三方主题的目录中。

```text
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```

最后就跟上面的操作一样，打开配置文件，把主题设置进去。

最后记得 source 一下。

```text
source ~/.zshrc
```

# 四、安装 powerline 和 PowerFonts

powerline 是 oh my zsh 依赖的一个插件。

这个插件主要解决很多关于 oh my zsh 主题中的字体问题。

当然，这个插件不一定要求装啊，如果你遇到有乱码问题，就需要装上了。

为什么会出现乱码的情况呢？

因为有些特殊的主题，有特殊的表情和符号。比如上面提到的 powerlevel9k。

废话不多说了，直接上官网。

```text
官网地址：https://powerline.readthedocs.io/en/latest/installation.html
```

如果你去看了 powerline 的官网，会发现 powerline 是用 python 写的，所以安装起来也很方便，只需要一条命令就好了。

```text
pip install powerline-status
```

当然安装之前要确保你已经安装了 python 环境和 pip , python 环境一般 Mac 系统都会自带的，所以如果你在安装过程中遇到：

```text
zsh: command not found: pip。
```

那就是 pip 没有安装了。

你也可以通过命令来安装。

```text
sudo easy_install pip
```

如果还有乱码，那是因为 PowerFonts 还没有安装。

PowerFonts 是一个字体库，要安装字体库需要先把 `git clone` 到本地，然后执行源码中的 `install.sh` 。

具体的流程如下：

```text
# git clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```



安装完成之后，就可以设置 iTerm2 的字体，具体的操作是 iTerm2 -> Preferences -> Profiles -> Text，在 Font 区域选中 Change Font，看个人喜欢，选择字体，字体名字带有 `for powerline` 的就不会乱码了。

**7、色彩预设**

itme2 支持各种色彩主题。

可以从官网那里看到各种介绍，这里就不再做详细的介绍了，主要还是讲下详细的步骤。

```text
官网地址：https://iterm2colorschemes.com/
```

你可以先把色彩主题文件下载下来。

在官网中直接点击图标下载

当然，也可以通过运行命令来下载

```text
git clone https://github.com/mbadolato/iTerm2-Color-Schemes ~/Downloads/itemcolor
```

注意，这里的 `~/Downloads/itemcolor` 是指放置下载文件的目录地址，可自行修改。

我是通过官网直接下载的，下载完成后，可以看到有很多文件夹，这里主要关注 `schemes` 和 `screenshots` 就好。

`schemes` 文件夹里主要是放置色彩主题文件的。

`screenshots` 则是各种色彩主题预设的预览图。

大家可以根据个人的喜好选自己喜欢的色彩主题，然后在 iterm2 中选择 Preference -> Profiles -> Colors ，导入色彩主题，并勾上就可以。

# 五、命令补全

zsh-autosuggestion 是一个 zsh 命令补全，提示的插件。

具体的流程如下：

```text
cd ~/.oh-my-zsh/custom/plugins/
git clone https://github.com/zsh-users/zsh-autosuggestions
vi ~/.zshrc
```

然后找到 `plugins` 把 `zsh-autosuggestions` 加上就行。

当然你也可以直接打开 `.zshrc` 这个文件，找到 `plugins` 把 `zsh-autosuggestions` 加上。

```bash
# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(zsh-autosuggestions git)
```

记得保存。

安装完成后，具体的效果如下，只要打 `op` ，它就会自动提示我之前打过的命令 `open ~/.zshrc`，非常方便。

像这种插件还有很多，比如 zsh-syntax-highlighting 语法高亮的插件，都非常拥有，这里就不一一介绍了。



# 六、zsh 常用快捷键

- CTRL + A
  ： 移动到行(line)的开始位置~~

- CTRL + E
  : 移动到行的最后面~~

- CTRL + [left arrow]
  : 向后移动一个单词(one word)~~

- CTRL + [right arrow]
  : 向前移动一个单词~~

- CTRL + U/Q
  : 将整行清除~~

- ESC + [backspack]
  : 删除光标前面的单词

- CTRL + W
  : 同上~~

- CTRL + D
  : 删除光标后面的字符

- CTRL + R
  : 搜索历史

- CTRL + _
  : 撤销最后一次的改变

- CTRL + L
  : 清空屏幕

- CTRL + S
  : 停止向屏幕继续输出

- !!
  : 执行历史记录中的上一个命令

- !abc
  : 打印历史记录中的以 abc开头的命令

- 命令ｒ　可自动执行上一条命令

- 命令!!可以回溯上一次的命令。

- dirs -v 查看历史访问目录

- - d dirs -v的简写
  - ~1 切换到第一个目录



- autojump 只能让你跳到那些你已经用 cd 到过的目录

- - jo dir 使用资源管理器打开指定dir
  - j dir 打开 dir



# 七、常用zsh git的快捷键

```bash
g - git
gst - git status
gl - git pull
gup - git pull --rebase
gp - git push
gd - git diff
gdc - git diff --cached
gdv - git diff -w "$@" | view
gc - git commit -v
gc! - git commit -v --amend
gca - git commit -v -a
gca! - git commit -v -a --amend
gcmsg - git commit -m
gco - git checkout
gcm - git checkout master
gr - git remote
grv - git remote -v
grmv - git remote rename
grrm - git remote remove
gsetr - git remote set-url
grup - git remote update
grbi - git rebase -i
grbc - git rebase --continue
grba - git rebase --abort
gb - git branch
gba - git branch -a
gcount - git shortlog -sn
gcl - git config --list
gcp - git cherry-pick
glg - git log --stat --max-count=10
glgg - git log --graph --max-count=10
glgga - git log --graph --decorate --all
glo - git log --oneline --decorate --color
glog - git log --oneline --decorate --color --graph
gss - git status -s
ga - git add
gm - git merge
grh - git reset HEAD
grhh - git reset HEAD --hard
gclean - git reset --hard && git clean -dfx
gwc - git whatchanged -p --abbrev-commit --pretty=medium
gsts - git stash show --text
gsta - git stash
gstp - git stash pop
gstd - git stash drop
ggpull - git pull origin $(current_branch)
ggpur - git pull --rebase origin $(current_branch)
ggpush - git push origin $(current_branch)
ggpnp - git pull origin $(current_branch) && git push origin $(current_branch)
glp - _git_log_prettily
```

zsh git的快捷命令 实现原理

实现原理 别名

```bash
alias gst='git status'
alias gp='git push'
alias gp='git push
```

[源码](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/git/git.plugin.zsh)



# 八、item2 安装 autojump

autojump 只能让你跳到那些你已经用 cd 打开过的目录。

[官方文档](https://github.com/wting/autojump)

```bash
brew install autojump
```

在 `.zshrc` 中找到 `plugins`，在后面添加

```bash
plugins=(git autojump)
```

然后继续在上述文件中添加

```bash
[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh
```

`source ~/.zshrc`



## 8.1 autojump 用法

[官方文档](https://github.com/wting/autojump#usage)

```bash
j -v # 查看安装的 autojump 的版本
j -h # 查看帮助选项
j [目录的名字或名字的一部分] // 不受当前所在目录的限制
j --stat # 查看每个文件夹的圈中和全部文件夹计算得出的总权重 的统计数据
j // 进入权重最高的目录

# 改变权重
j -i [权重] // 增加
j -d [权重] // 减少

j --purge // 去除不存在的路径
jco -c // 在文件管理器中打开一个子目录
```

