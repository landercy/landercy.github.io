# AZURE CLI

Azure 区分 azure.com 和 azure.cn，当使用 `az login`时，默认会登录到 azure.com。
当需要登录到 azure.cn 时，需要先执行 `az cloud set --name AzureChinaCloud`

## ACCOUNT

```bash
# 查看当前订阅
az account show

# 查看授权的订阅列表
az account list

# 切换当前订阅
az account set -s $SUBSCRIPTION_NAME
```

## ACR

```
az acr list | grep name
az acr show-usage -n <registry name>
az acr repository list -n <registry name>
az acr repository show -n <registry name> --repository <repository>| grep -i count
az acr repository show-tags -n <registry name> --repository <repository>
az acr repository delete -n <registry name> --repository <repository>
```

## EXTENSION

```bash
# 查看已安装的扩展
az extension list

# 安装扩展
az extension add --upgrade --name azure-iot

# 更新扩展
az extension update --name azure-iot
```

## IOT

```bash
# 查看授权的IoThub
az iot hub list 

# 查看连接字符串
az iot hub connection-string show

# 注册设备
az iot hub device-identity create \
  -l $IOTHUB_CONNECTION_STRING \
  --device-id $DEVICE_ID
  
# 查询设备信息
az iot hub device-twin show \
  -l $IOTHUB_CONNECTION_STRING \
  -d $DEVICE_ID
  
# 查询设备密钥
az iot hub device-identity connection-string show \
  -l $IOTHUB_CONNECTION_STRING \
  -d $DEVICE_ID

# 监听指定设备遥测消息
az iot hub monitor-events \
  -l $IOTHUB_CONNECTION_STRING \
  -d $DEVICE_ID
  
# 发送遥测消息
az iot device send-d2c-message \
  -l $IOTHUB_CONNECTION_STRING \
  -d $DEVICE_ID \
  --data $DATA
  
# 执行直接方法
az iot hub invoke-device-method \
  -l $IOTHUB_CONNECTION_STRING \
  -d $DEVICE_ID \
  --mn $METHOD \
  --mp $PAYLOAD
```

# DATABRICKS

## MOUNT DATALAKE

```py
tenant_id = ''
client_id = ''
client_secret = ''
client_endpoint = 'https://login.partner.microsoftonline.cn/' + tenant_id + '/oauth2/token'

storage_account_name = ''
storage_container_name = ''
storage_account_domain = storage_account_name + '.dfs.core.chinacloudapi.cn'
storage_uri = 'abfss://' + storage_container_name + '@' + storage_account_domain,

mount_path = '/mnt/DIR_NAME'

configs = {
  'fs.azure.account.auth.type': 'OAuth',
  'fs.azure.account.oauth.provider.type': 'org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider',
  'fs.azure.account.oauth2.client.id': client_id,
  'fs.azure.account.oauth2.client.secret': client_secret,
  'fs.azure.account.oauth2.client.endpoint': client_endpoint,
  'fs.azure.createRemoteFileSystemDuringInitialization': 'true'
}

dbutils.fs.mount(source = storage_uri, mount_point = mount_path, extra_configs = configs)
```
