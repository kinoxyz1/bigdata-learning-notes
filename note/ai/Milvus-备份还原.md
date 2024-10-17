






[下载工具](https://github.com/zilliztech/milvus-backup/releases)

[下载配置文件](https://raw.githubusercontent.com/zilliztech/milvus-backup/master/configs/backup.yaml)

```bash
├── milvus-backup 
├────── configs 
└────────── backup.yaml
```

backup.yaml 说明:
```bash
# Configures the system log output.
log:
  level: info # Only supports debug, info, warn, error, panic, or fatal. Default 'info'.
  console: true # whether print log to console
  file:
    rootPath: "logs/backup.log"

http:
  simpleResponse: true

# milvus proxy address, compatible to milvus.yaml
milvus:
  address: localhost   # Milvus ip
  port: 19530          # Milvus 端口
  authorizationEnabled: false
  # tls mode values [0, 1, 2]
  # 0 is close, 1 is one-way authentication, 2 is two-way authentication.
  tlsMode: 0
  user: "root"
  password: "Milvus"

# Related configuration of minio, which is responsible for data persistence for Milvus.
minio:
  # Milvus storage configs, make them the same with milvus config
  storageType: "minio" # support storage type: local, minio, s3, aws, gcp, ali(aliyun), azure, tc(tencent)
  address: localhost # Address of MinIO/S3
  port: 9000   # Port of MinIO/S3
  accessKeyID: minioadmin  # accessKeyID of MinIO/S3
  secretAccessKey: minioadmin # MinIO/S3 encryption string
  useSSL: false # Access to MinIO/S3 with SSL
  useIAM: false
  iamEndpoint: ""
  # 备份时: 待备份的Milvus所在的minio的Bucket名字, 比如这里是 Milvus Operator 安装的集群,就是 milvus-release
  # 还原时: 待还原的Milvus所在的minio的Bucket名字, 比如这里是 docker 安装的集群,就是 a-bucket
  bucketName: "milvus-release" 
  rootPath: "files" # Milvus storage root path in MinIO/S3, make it the same as your milvus instance

  # Backup storage configs, the storage you want to put the backup data
  backupStorageType: "minio" # support storage type: local, minio, s3, aws, gcp, ali(aliyun), azure, tc(tencent)
  backupAddress: localhost # Address of MinIO/S3
  backupPort: 9000   # Port of MinIO/S3
  backupAccessKeyID: minioadmin  # accessKeyID of MinIO/S3
  backupSecretAccessKey: minioadmin # MinIO/S3 encryption string
  # 备份时: 备份到指定的 Bucket 中, 这里是对应 Bucket 的名字
  # 还原时: 要还原的数据所在的Minio的Bucket的名字
  backupBucketName: "milvus-release" 
  backupRootPath: "backup" # Rootpath to store backup data. Backup data will store to backupBucketName/backupRootPath

  # If you need to back up or restore data between two different storage systems, direct client-side copying is not supported.
  # Set this option to true to enable data transfer through Milvus Backup.
  # Note: This option will be automatically set to true if `minio.storageType` and `minio.backupStorageType` differ.
  # However, if they are the same but belong to different services, you must manually set this option to `true`.
  crossStorage: "false"

backup:
  maxSegmentGroupSize: 2G

  parallelism:
    # collection level parallelism to backup
    backupCollection: 4
    # thread pool to copy data. reduce it if blocks your storage's network bandwidth
    copydata: 128
    # Collection level parallelism to restore
    restoreCollection: 2

  # keep temporary files during restore, only use to debug
  keepTempFiles: false

  # Pause GC during backup through Milvus Http API.
  gcPause:
    enable: true
    seconds: 7200
    address: http://localhost:9091
```

由于 Milvus Backup 无法将数据备份到本地路径，因此在定制配置文件时要确保 Minio 设置正确。

> 默认 Minio 存储桶的名称因安装 Milvus 的方式而异。更改 Minio 设置时，请参阅下表。

| 字段	|  Docker Compose	| Helm / Milvus Operator |
| --- | --- | --- |
| bucketName | 	a-bucket	 | milvus-release |
| rootPath | 	文件	 | 文件 |


# 备份
```bash
./milvus-backup create -n <backup_name>
```


# 还原
```bash
./milvus-backup restore -n my_backup -s _recover
```

请注意，上述脚本假定您已运行带有-s 标志的restore 命令，且后缀设置为-recover 。请根据需要对脚本进行必要的修改。


