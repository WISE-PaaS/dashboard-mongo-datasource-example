# MongoDB Datasource Usage Example for WISE-PaaS/Dashboard

## Features

Allows MongoDB to be used as a data source for Grafana by providing a proxy to convert the Grafana Data source [API](http://docs.grafana.org/plugins/developing/datasources/) into MongoDB aggregation queries.

Credits go to the project at https://github.com/JamesOsgood/mongodb-grafana

**The codes used in this example comes from an open source project, Advantech will not be responsible for any issues caused by the use of these codes.**

## Requirements

- **Grafana** > 3.x.x
- **MongoDB** > 3.4.x
- **WISE-PaaS/EnSaaS account**
- **WISE-PaaS/Dashboard** > 1.3.x

## Installation

### Install the Grafana plugin components

- WISE-PaaS/Dashboard already included this MongoDB datasource plugin in the distribution, so there's no need to install the datasource plugin.

### Install and Start the MongoDB proxy server

#### 1. Create secret in Portal-Service

- Create Database in Cluster

  <img src="src/img/create secret.png" alt="sample_secret" style="width: 500px;"/>
  
  <img src="src/img/mongodb-secret.png" alt="sample_secret" style="width: 500px;"/>

#### 2. Check yaml file

- Replace the docker account with your own account

  <img src="src/img/sample_dockeraccount.png" alt="sample_secret" style="width: 500px;"/>
  
- Replace secret

  <img src="src/img/1__WISE-PaaS___Overview.png" alt="sample_secret" style="width: 500px;"/>
  
#### 3. Check ingress file
  
- Replace space name with your own space

  <img src="src/img/ingress_yaml_—_mongo-api-dashboard.png" alt="sample_secret" style="width: 500px;"/>

#### 4. Package to Dockerhub

- `docker build -t {docker account/mongodb:api} .`

  -t : Specify the name of the target image to be created
  
  "." : Location of the directory containing the Dockerfile
- `docker push {docker account/mongodb:api}`

#### . Apply to WISE-PaaS

- `kubectl apply -f k8s/`

## Add data source (MongoDB) in Dashboard

- View MongoDB-API Domain url in ingress and copy that in HTTP

<img src="src/img/view secret.png" alt="sample_datasource" style="width: 500px;"/>

<img src="src/img/mongodb-api-url.png" alt="sample_datasource" style="width: 500px;"/>

- View MongoDB secret in Portal-Service and copy that in MongoDB details

<img src="src/img/credentials.png" alt="sample_datasource" style="width: 500px;"/>

<img src="src/img/data source.png" alt="sample_datasource" style="width: 500px;"/>

Then save the data source

#### Example 1 - Simple aggregate to rename fields

<img src="src/img/sample_example1.png" alt="sample_example1" style="width: 500px;"/>

This should show a graph of data values from a dummy data set, which has the document format like below :

<img src="src/img/sample_robo3T.png" alt="sample_robo3t" style="width: 500px;"/>

Clicking on the title of the graph allows you to see the aggregation query being run against the data source

<img src="src/img/grafana_mongo_demo_5.PNG" alt="Sample Query" style="width: 800px;"/>

The query here is

```javascript
db.default.aggregate([{ $match: { ts: { $gte: '$from', $lt: '$to' } } }]);
```

The API is expecting back documents with the following fields

- `name` - Name of the series ( will be displayed on the graph)
- `value` - The float value of the point
- `ts` - The time of the point as a BSON date

These documents are then converted into the [Grafana API](http://docs.grafana.org/plugins/developing/datasources/)

`$from` and `$to` are expanded by the plugin as BSON dates based on the range settings on the UI.

#### Example 2 - Using a Tabel Panel
<img src="src/img/sample_table.png" alt="sample_table.PNG" style="width: 800px;"/>

Table panels are now supported with queries of the form

```javascript
db.default.aggregate([
  { $match: { ts: { $gte: '$from', $lt: '$to' } } },
  { $group: { _id: '$machine', total: { $avg: '$value' }, ts: { $max: '$ts' } } }
]);
```

This should show a graph of data values from a default data set, which has the document format like below :

<img src="src/img/sample_tobo3t_datasource_format.png" alt="sample_tobo3t_datasource_format" style="width: 500px;"/>
