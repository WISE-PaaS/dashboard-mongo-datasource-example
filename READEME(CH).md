# MongoDB Datasource for WISE-PaaS/Dashboard 使用範例

## 特色

允許 MongoDB 用作 Grafana 的數據源 [API](http://docs.grafana.org/plugins/developing/datasources/)

信賴的項目 <https://github.com/JamesOsgood/mongodb-grafana>

## 版本需求

- **Grafana** > 3.x.x
- **MongoDB** > 3.4.x
- **WISE-PaaS / EnSaaS account**
- **WISE-PaaS / Dashboard** > 1.3.x

## 安裝

### 安裝 Grafana 插件當中的組件

- WISE-PaaS / Dashboard 已在發行版中包含此 MongoDB 數據源插件，因此無需安裝數據源插件。

### 安裝並啟動 MongoDB 代理服務器

#### 1. 在 Portal-Service 創建 secret

- 創建 Database 在 Cluster 當中

  <img src="src/img/create secret.png" alt="sample_secret" style="width: 500px;"/>
  
  <img src="src/img/mongodb-secret.png" alt="sample_secret" style="width: 500px;"/>

#### 2. 檢查 yaml 檔案

- 將 docker 帳號更換成自己的帳號

  <img src="src/img/sample_dockeraccount.png" alt="sample_secret" style="width: 500px;"/>
- 更換 secret
  <img src="src/img/sample_secret.png" alt="sample_secret" style="width: 500px;"/>
  
#### 3. Check ingress file
  
- 更換為自己的 Space 名稱 

  <img src="src/img/ingress_yaml_—_mongo-api-dashboard.png" alt="sample_secret" style="width: 500px;"/>

#### 4. 打包到 Dockerhub 上

- `docker build -t {docker account/mongodb:api} .`\
  -t : 指定要創建的目標鏡像名稱\
  "." : DockerFile 文件所在目錄，也可以指定 DockerFile 絕對路徑
- `docker push {docker account/mongodb:api}`

#### 5. Apply 到 WISE-PaaS 雲端

- `kubectl apply -f k8s/`

## 添加數據源(MongoDB)在 Dashboard

- 複製 Domain MongoDB API URL 到 HTTP 當中

<img src="src/img/view secret.png" alt="sample_datasource" style="width: 500px;"/>

<img src="src/img/mongodb-api-url.png" alt="sample_datasource" style="width: 500px;"/>

- 複製 secret 當中的 credentials 到 MongoDB details 當中

<img src="src/img/credentials.png" alt="sample_datasource" style="width: 500px;"/>

<img src="src/img/data source.png" alt="sample_datasource" style="width: 500px;"/>

然後保存數據源

#### 範例 1 - 使用時序面板

<img src="src/img/sample_example1.png" alt="sample_example1" style="width: 500px;"/>

```javascript
db.default.aggregate([{ $match: { ts: { $gte: '$from', $lt: '$to' } } }]);
```

數據來源是 default 數據，其格式如下：

<img src="src/img/sample_robo3T.png" alt="sample_robo3t" style="width: 500px;"/>

API 要求返回具有以下字段的文檔

- `name` - 名稱系列 ( 將顯示在圖表上 )
- `value` - 點的浮點值
- `ts` - 點的時間作為 BSON 日期

然後將這些文檔轉換為 [Grafana API](http://docs.grafana.org/plugins/developing/datasources/)

插件會將`$ from`和`$ to`擴展為 BSON 日期（基於 UI 上的範圍設置）

#### 範例 2 - 使用表格面板

<img src="src/img/sample_table.png" alt="sample_table" style="width: 500px;"/>

目前支持表格查詢方式

```javascript
db.default.aggregate([
  { $match: { ts: { $gte: '$from', $lt: '$to' } } },
  { $group: { _id: '$machine', total: { $avg: '$value' }, ts: { $max: '$ts' } } }
]);
```

數據來源是 default 數據，其數據內容格式如下：

<img src="src/img/sample_tobo3t_datasource_format.png" alt="sample_tobo3t_datasource_format" style="width: 500px;"/>
