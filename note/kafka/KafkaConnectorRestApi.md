


---
# 一、示例
## 1.1 提交
```bash
curl -s -X POST -H 'Content-Type: application/json' --data @ConnectorsConfig.json http://localhost:8083/connectors
```

## 1.2 查看活动的 connectors
```bash
curl localhost:8083/connectors
```

## 1.3 查看指定的 connectors 信息
```bash
curl localhost:8083/connectors/connectorsName
```

## 1.4 查看指定的 connectors 状态
```bash
curl localhost:8083/connectors/connectorsName/status
```

## 1.5 暂停一个 connectors
```bash
curl -X PUT localhost:8083/connectors/connectorsName/paus
```

## 1.6 唤醒一个 connectors
```bash
curl -X PUT localhost:8083/connectors/connectorsName/resume
```

## 1.7 重启一个 connectors
```bash
curl -X POST localhost:8083/connectors/connectorsName/restart
```

## 1.8 删除一个 connectors
```bash
curl -X DELETE localhost:8083/connectors/connectorsName
```

# 二、官网
http://kafka.apache.org/documentation/#connect_rest