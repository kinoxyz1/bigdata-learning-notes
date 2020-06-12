![在这里插入图片描述](../../img/sentry/sentry异常1/20200521095639179.png)

这个原因是这个用户没有查看的权限，hive，hue就可以，因为sentry中没有配置default组为超级组
![在这里插入图片描述](../../img/sentry/sentry异常1/2020052114162956.png)


在hue中创建hive账号、hive组，使用该账号登陆, 即可添加角色
![在这里插入图片描述](../../img/sentry/sentry异常1/20200521141812599.png)