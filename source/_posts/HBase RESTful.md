---
title: HBase RESTful API
tags: 大数据
categories: bigdata
---
## 启动 HBase RESTful 服务
hbase rest api 服务启动：
```bash
hbase-daemon.sh start rest
```

Follow these instructions for each HBase host fulfilling the REST server role.

* To start the REST server as a foreground process, use the following command:
```bash
$ bin/hbase rest start
```
* To start the REST server as a background process, use the following command:
```bash
$ bin/hbase-daemon.sh start rest
```
* To use a different port than the default of 8080, use the -p option.
* To stop a running HBase REST server, use the following command:
```bash
$ bin/hbase-daemon.sh stop rest
```


## Cluster-Wide Endpoints
Endpoint	HTTP Verb	Description	Example
__/version/cluster__	GET	Version of HBase running on this cluster
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:8080/version/cluster"
```         
__/status/cluster__	GET	Cluster status
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:8080/status/cluster"
```         
__/__	GET	List of all nonsystem tables
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:8080/"
```

## Table Endpoints
Endpoint	HTTP Verb	Description	Example
__/table/schema__	GET	Describe the schema of the specified table.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/schema"
```         
__/table/schema__	POST	Create a new table, or replace an existing table's schema with the provided schema
```bash
curl -vi -X POST \
         -H "Accept: text/xml" \
         -H "Content-Type: text/xml" \
         -d '<?xml version="1.0" encoding="UTF-8"?><TableSchema name="users"><ColumnSchema name="cf" /></TableSchema>' \
         "http://example.com:20550/users/schema"
```         
__/table/schema__	UPDATE	Update an existing table with the provided schema fragment
```bash
curl -vi -X PUT \
         -H "Accept: text/xml" \
         -H "Content-Type: text/xml" \
         -d '<?xml version="1.0" encoding="UTF-8"?><TableSchema name="users"><ColumnSchema name="cf" KEEP_DELETED_CELLS="true" /></TableSchema>' \
         "http://example.com:20550/users/schema"
```         
__/table/schema__	DELETE	Delete the table. You must use thetable/schemaendpoint, not just table/.
```bash
curl -vi -X DELETE \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/schema"
```         
__/table/regions__	GET	List the table regions.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/regions"
```

## Endpoints for Get Operations
Endpoint	HTTP Verb	Description	Example
__/table/row/column:qualifier/timestamp__	GET	Get the value of a single row. Values are Base-64 encoded.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/row1"
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/row1/cf:a/1458586888395"
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/row1/cf:a"
curl -vi -X GET \
         -H "Accept: text/xml" \
          "http://example.com:20550/users/row1/cf:a/"
```          
__/table/row/column:qualifier?v=number_of_versions__	 	Multi-Get a specified number of versions of a given cell. Values are Base-64 encoded.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/row1/cf:a?v=2"
```

## Endpoints for Scan Operations
Endpoint	HTTP Verb	Description	Example
__/table/scanner/__	PUT	Get a Scanner object. Required by all other Scan operations. Adjust the batch parameter to the number of rows the scan should return in a batch. See the next example for adding filters to your Scanner. The scanner endpoint URL is returned as the Location in the HTTP response. The other examples in this table assume that the Scanner endpoint ishttp://example.com:20550/users/scanner/145869072824375522207.
```bash
curl -vi -X PUT \
         -H "Accept: text/xml" \
         -H "Content-Type: text/xml" \
         -d '<Scanner batch="1"/>' \
         "http://example.com:20550/users/scanner/"
```         
__/table/scanner/__	PUT	To supply filters to the Scanner object or configure the Scanner in any other way, you can create a text file and add your filter to the file. For example, to return only rows for which keys start with u123and use a batch size of 100:
```
<Scanner batch="100">
  <filter>
    {
      "type": "PrefixFilter",
      "value": "u123"
    }
  </filter>
</Scanner>
```
Pass the file to the -d argument of the curl request.
```bash
curl -vi -X PUT \
         -H "Accept: text/xml" \
         -H "Content-Type:text/xml" \
         -d @filter.txt \
         "http://example.com:20550/users/scanner/"
```         
__/table/scanner/scanner_id__	GET	Get the next batch from the scanner. Cell values are byte-encoded. If the scanner is exhausted, HTTP status 204 is returned.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/scanner/145869072824375522207"
```         
__/table/scanner/scanner_id__	DELETE	Deletes the scanner and frees the resources it was using.
```bash
curl -vi -X DELETE \
         -H "Accept: text/xml" \
         "http://example.com:20550/users/scanner/145869072824375522207"
```

## Endpoints for Put Operations
Endpoint	HTTP Verb	Description	Example
__/table/row_key/__	PUT	Write a row to a table. The row, column qualifier, and value must each be Base-64 encoded. To encode a string, you can use the base64command-line utility. To decode the string, usebase64 -d. The payload is in the --data argument, so the/users/fakerowvalue is a placeholder. Insert multiple rows by adding them to the<CellSet>element. You can also save the data to be inserted to a file and pass it to the -dparameter with the syntax -d @filename.txt.	XML:
```bash
curl -vi -X PUT \
         -H "Accept: text/xml" \
         -H "Content-Type: text/xml" \
         -d '<?xml version="1.0" encoding="UTF-8" standalone="yes"?><CellSet><Row key="cm93NQo="><Cell column="Y2Y6ZQo=">dmFsdWU1Cg==</Cell></Row></CellSet>' \
         "http://example.com:20550/users/fakerow"
curl -vi -X PUT \
         -H "Accept: text/json" \
         -H "Content-Type: text/json" \
         -d '{"Row":[{"key":"cm93NQo=", "Cell": [{"column":"Y2Y6ZQo=", "$":"dmFsdWU1Cg=="}]}]}' \
         "example.com:20550/users/fakerow"
```

## Namespace Endpoints
Endpoint	HTTP Verb	Description	Example
__/namespaces__	GET	List all namespaces.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/namespaces/"
```         
__/namespaces/namespace__	GET	Describe a specific namespace.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/namespaces/special_ns"
```         
__/namespaces/namespace__	POST	Create a new namespace.
```bash
curl -vi -X POST \
         -H "Accept: text/xml" \
         "example.com:20550/namespaces/special_ns"
```         
__/namespaces/namespace/tables__	GET	List all tables in a specific namespace.
```bash
curl -vi -X GET \
         -H "Accept: text/xml" \
         "http://example.com:20550/namespaces/special_ns/tables"
```         
__/namespaces/namespace__	PUT	Alter an existing namespace. Currently not used.
```bash
curl -vi -X PUT \
         -H "Accept: text/xml" \
         "http://example.com:20550/namespaces/special_ns"
```         
__/namespaces/namespace__	DELETE	Delete a namespace. The namespace must be empty.
```bash
curl -vi -X DELETE \
         -H "Accept: text/xml" \
         "example.com:20550/namespaces/special_ns"
```
