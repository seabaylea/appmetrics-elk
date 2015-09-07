# ELK Connector for Node Application Metrics

A connector that collects data using 'appmetrics' and sends it to a configured ElasticSearch instance in LogStash format for use with Kibana.

## Getting Started

### Installation
The ELK Connector for Node Application Metrics can be installed via `npm`:
```sh
$ npm install appmetrics-elk
```
This is designed to be used with an [ElasticSearch][1] database, and a visualization tool such as [Kibana][2].

### Configuring the ELK Connector for Node Application Metrics 

The connector can be used in your application by requiring it as the first line of your application:
```js
var appmetrics = require('appmetrics-elk').monitor();
```
This will send all of the available `appmetrics` data to the ElasticSearch instance, as well as returning an appmetrics object that can be used to control data collection.

```js
var appmetrics = require('appmetrics-elk').monitor();
appmetrics.disable('mysql');            // disable MySQL monitoring
```

Additionally, the `monitor()` API call can be passed an optional [ElasticSearch Configuration][3] object to configure the ElasticSearch connection, including database location and security.  
```js
var config = {
    hosts: [
        'https://es1.bluemix.net',
        'https://es2.bluemix.net'
    ],
    ssl: {
        ca: fs.readFileSync('./cacert.pem'),
        rejectUnauthorized: true
    }
}

var appmetrics = require('appmetrics-elk').monitor(config);
appmetrics.disable('mysql');            // disable MySQL monitoring
```

### Data Provided to ElasticSearch

The ELK Connector for Node Application Metrics uploads its data to the 'appmetrics' index in ElasticSearch. It sends the following values to ElasticSearch for every monitoring entry:

 Value                   | Description
:------------------------|:-------------------------------------------
 timestamp               | The time when the monitoring event occurred
 hostName                | The hostname for the machine the monitored process is running on
 pid                     | The process ID for the monitored process
 
Additional data is then included depending on the monitoring event.

**CPU Data**  

 Value                   | Description
:------------------------|:-------------------------------------------
 cpu.process             | The CPU usage of the application as a percentage of total machine CPU
 cpu.system              | The CPU usage of the system as a percentage of total machine CPU

**Memory Data**

 Value                   | Description
:------------------------|:-------------------------------------------
 memory.process.private  | The memory used by the application that cannot be shared with other processes, in bytes
 memory.process.physical | The RAM used by the application in bytes
 memory.process.virtual  | The memory address space used by the application in bytes
 memory.system.physical  | The total amount of RAM in use on the system in bytes
 memory.system.total     | The total amount of RAM available on the system in bytes

**Garbage Collection Data**  

 Value                   | Description
:------------------------|:-------------------------------------------
 gc.used                 | The JavaScript heap used by the application in bytes
 gc.size                 | The size of the JavaScript heap in bytes
 gc.type                 | The type of GC cycle, either 'M' or 'S'
 gc.duration             | The duration of the GC cycle in milliseconds
 
**HTTP Request Data**  

 Value                   | Description
:------------------------|:-------------------------------------------
 http.method             | The HTTP method used for the request
 http.url                | The URL on which the request was made
 http.duration           | The time taken for the HTTP request to be responded to in ms 

**MongoDB Query Data**
  
 Value                   | Description
:------------------------|:-------------------------------------------
 mongo.query             | The query made of the MongoDB database
 mongo.duration          | The time taken for the MongoDB query to be responded to in ms 
 
 **MySQL Query Data**  
 
 Value                   | Description
:------------------------|:-------------------------------------------
 mysql.query             | The query made of the MySQL database
 mysql.duration          | The time taken for the MySQL query to be responded to in ms 

<a name="custom-data"></a>
### Sending Custom Data to ElasticSearch
The Node Application Metrics to ELK Connector registers for events that it is aware of, and forwards the data from those events to ElasticSearch. The registration for those events is based on the 'mappings' files in the following directory:
```sh
node_modules/appmetrics-elk/mappings/
```
Any mappings files found in that directory are both used to configure how ElasticSearch handles the data, and to configure the monitoring events that are forwarded.

The `type` field is used to determine the name of the event to register for, and the `properties` fields are used to determine the values to send. Note that the values in the properties entry in the mapping file must match the fields in the monitoring event data. For example, the CPU event has the following data:
* `process`
* `system`

The mapping file that causes this data to be sent to ElasticSearch therefore has the following structure:
```json
{
    "index":  "appmetrics",
    "type":   "cpu",
    "body": {
        "_source" : {"compress" : true},
        "_ttl" : {"enabled" : true, "default" : "90d"},
        "properties": {
            "timestamp":    {"type": "date", "format": "dateOptionalTime"},
            "hostName":     {"type": "string", "index": "not_analyzed"},
            "pid":          {"type": "integer"},
            "cpu": {
                "type": "nested",
                "include_in_parent": true,
                "properties": {            
                    "process":      {"type": "float"},
                    "system":       {"type": "float"}
                }
            }
        }   
    }
}
```
This causes the Node Application Metrics to ELK Connector to register for `cpu` events and forward the `process` and `system` values as `cpu.process` and `cpu.system`.

## Using Kibana with the ELK Connector for Node Application Metrics 
During startup the ELK Connector for Node Application Metrics attempts to provide some pre-configuration for using Kibana 4 with the provided data. It does this by uploading the following if there are not existing ones already associated with the 'appmetrics' index:
* An index pattern
* Data mappings for the data types
* Default visualizations for the data types
* A default dashboard

Each of these configurations are dynamically loaded from the 'indexes', 'mappings', 'charts' and 'dashboards' directories in the `appmetrics-elk` install directory. It is therefore possible to prevent the configurations from being automatically added by deleting those files, or to add to them by adding existing files.  

**Note:** The 'mappings' directory also provides the configuration of which types of monitoring data are uploaded to ElasticSearch so entries should only be deleted if necessary. See *[Sending Custom Data to ElasticSeach](#custom-data)* for more information.

### Visualizing the data with Kibana 4
The pre-configuration for Kibana 4 provdes a number of default visualizations, as well as a default dashboard. These can subsequently be modified or new visualizations and dashboards created.

**Using the dashboard**  
In order to avoid replacing any dashboard already in use in Kibana 4 with the one supplied by the Node Application Metrics to ELK Connector, the dashboard is made available but not loaded. The dashboard is loaded by:  
1. Click on the "Dashboard" tab  
2. Select the "Load Saved Dashboard" icon  
3. Select "Default AppMetrics Dashboard" from the list of saved dashboards  
This now loads a simple dashboard that uses some of the default visualization charts provided by the Node Application Metrics to ELK Connector.

**Using the visualization charts**  
In addition to the dashboard, a number of visualization charts are also provided. These can be added to a dashboard using the following steps:  
1. Click on the "Dashboard" tab  
2. Click on the "Add Visualization" icon  
3. Select a visualization chart from the menu  
4. Place and resize the visualization chart by dragging it across the screen  

You can also create your own charts using the "Visualize" tab.

### License
The Node Application Metrics to ELK Connector is licensed using an Apache v2.0 License.

### Version
1.0.0

[1]:https://www.elastic.co/downloads/elasticsearch
[2]:https://www.elastic.co/downloads/kibana
[3]:https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html
