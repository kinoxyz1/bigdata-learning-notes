










设置环境变量
```bash
$ echo $SSLKEYLOGFILE
/Users/kino/Downloads/sslkeylog.log
```

wireshark 点 设置 -> Protocols -> TLS -> (Pre)-Master-Secret log filename 这个填 `$SSLKEYLOGFILE` 的值, 然后重启 wireshark.

再重启 Chrome， Macbook 可以这样启动:
```bash
$ open -a "Google Chrome"
```









