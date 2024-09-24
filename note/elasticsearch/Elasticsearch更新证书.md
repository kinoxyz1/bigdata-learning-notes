





更新证书有两种方式:
1. 使用不同的 CA 更新安全证书
2. 使用相同的 CA 更新安全证书

详情可以看官方文档: https://www.elastic.co/guide/en/elasticsearch/reference/current/security-ssl.html#updating-certificates

这里记录第一种方式(使用不同的 CA 更新安全证书)


# 生成 CA 证书
```bash
$ ./bin/elasticsearch-certutil ca -ip xxx.xxx.xxx.xxx
1. 出现提示时，接受默认文件名，即elastic-stack-ca.p12 。该文件包含您的 CA 的公共证书以及用于为每个节点签署证书的私钥。
2. 输入您的 CA 的密码。如果您不部署到生产环境，则可以选择将密码留空。
```
[elasticsearch-certutil 的参数说明](https://www.elastic.co/guide/en/elasticsearch/reference/current/certutil.html#certutil-cert)

如果既未指定 IP 地址也未指定 DNS 名称，Elastic Stack 产品无法执行主机名验证，您可能需要将verification_mode安全设置配置为仅certificate 。



# 生成证书和私钥
```bash
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```
- --ca <ca_file>: 用于签署证书的 CA 文件的名称。 elasticsearch-certutil工具的默认文件名是elastic-stack-ca.p12 。
    - 输入 CA 的密码，或者如果您在上一步中未配置密码，请按Enter 。
    - 为证书创建密码并接受默认文件名。 输出文件是一个名为elastic-certificates.p12的密钥库。该文件包含节点证书、节点密钥和 CA 证书。

将elastic-certificates.p12文件复制到$ES_PATH_CONF目录。

为集群中的每个节点完成以下步骤。要加入同一集群，所有节点必须共享相同的cluster.name值。

1. 打开 $ES_PATH_CONF/elasticsearch.yml 文件并进行以下更改：
```bash
# 1. 添加cluster-name设置并输入集群的名称：
cluster.name: my-cluster
# 2. 添加node.name设置并输入节点的名称。 Elasticsearch 启动时，节点名称默认为机器的主机名。
node.name: node-1
# 3. 添加以下设置以启用节点间通信并提供对节点证书的访问。
## 由于您在集群中的每个节点上使用相同的elastic-certificates.p12文件，因此将验证模式设置为certificate ：
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.client_authentication: required
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
# 如果要使用主机名验证，请将验证模式设置为full 。您应该为与 DNS 或 IP 地址匹配的每个主机生成不同的证书。请参阅 xpack.security.transport.ssl.verification_mode TLS 设置中的参数。
```
2. 如果您在创建节点证书时输入了密码，请运行以下命令将密码存储在Elasticsearch密钥库中：
> ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password

> ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password

3. 为集群中的每个节点完成前面的步骤。

4. 在集群中的每个节点上启动 Elasticsearch。启动和停止Elasticsearch 的方法根据您的安装方式而有所不同。

# 加密 Elasticsearch 的 HTTP 客户端通信

```bash
./bin/elasticsearch-certutil http
```
此命令生成一个.zip文件，其中包含用于 Elasticsearch 和 Kibana 的证书和密钥。每个文件夹都包含一个README.txt解释如何使用这些文件。

1. 当询问您是否要生成 CSR 时，请输入n 。
2. 当询问您是否要使用现有 CA 时，请输入y 。
3. 输入您的 CA 的路径。这是您为集群生成的elastic-stack-ca.p12文件的绝对路径。
4. 输入您的 CA 的密码。
5. 输入证书的到期值。您可以输入以年、月或日为单位的有效期。例如，输入90D表示 90 天。
6. 当询问您是否要为每个节点生成一个证书时，请输入y 。

   每个证书都有自己的私钥，并将针对特定主机名或 IP 地址颁发。

7. 出现提示时，输入集群中第一个节点的名称。使用生成节点证书时使用的相同节点名称。
8. 输入用于连接到第一个节点的所有主机名。这些主机名将作为 DNS 名称添加到证书的使用者备用名称 (SAN) 字段中。

    列出用于通过 HTTPS 连接到集群的每个主机名和变体。

9. 输入客户端可用于连接到您的节点的 IP 地址。
10. 对集群中的每个其他节点重复这些步骤。


为每个节点生成证书后，出现提示时输入私钥密码。

解压生成的elasticsearch-ssl-http.zip文件。此压缩文件包含 Elasticsearch 和 Kibana 的一个目录。
```bash
/elasticsearch
|_ README.txt
|_ http.p12
|_ sample-elasticsearch.yml

/kibana
|_ README.txt
|_ elasticsearch-ca.pem
|_ sample-kibana.yml
```
在集群中的每个节点上，完成以下步骤：

1. 将相关的http.p12证书复制到$ES_PATH_CONF目录中。
2. 编辑elasticsearch.yml文件以启用HTTPS 安全并指定http.p12安全证书的位置。

```bash
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: http.p12
```
3. 将私钥的密码添加到 Elasticsearch 中的安全设置中。

```bash
./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
```
4. 启动 Elasticsearch。



# 注意
如果在生成 CA证书时，没有加 ip 参数, 在kibana配置中需要加配置:
```bash
elasticsearch.ssl.verificationMode: certificate
```













