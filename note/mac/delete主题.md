









```bash
cd ~/Library/Preferences/


# 备份一下，然后开始详细操作：
1、用plutil -convert xml1 com.googlecode.iterm2.plist命令把plist转换成xml格式的纯文本
2、用编辑器打开文件，删除<dict></dict> 标签
3、用plutil -convert binary1 com.googlecode.iterm2.plist命令把plist转换成二进制文件
4、用修改后的文件替换掉原文件。重启iTerm2即可。
```