# Not Analyzed & Multi-Fields Exercise #

* Login into you virtual box
* Delete previously created index and its mapping:
```
curl -XDELETE 'localhost:9200/orders?pretty=true'
```
* Post new document:
```
curl -XPOST 'localhost:9200/orders/orders/1?pretty=true' 
  -H 'content-type: application/json' \
  -d '{"id": "1", "placedOn": "2016-10-17T13:03:30.830Z"}'
```
* Fetch mapping:
```
curl 'localhost:9200/orders/orders/_mapping?pretty=true'
```
* Expected response:
```
{
  "orders" : {
    "mappings" : {
      "orders" : {
        "properties" : {
          "id" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "placedOn" : {
            "type" : "date"
          }
        }
      }
    }
  }
}
```
* Add mapping for a new field
```
curl -XPUT 'localhost:9200/orders/orders/_mapping?pretty=true' \
  -H 'content-type: application/json' \
  -d '
{
  "orders" : {
    "properties" : {
      "id" : {
        "type" : "text"
      },
      "placedOn" : {
        "type" : "date",
        "format" : "strict_date_optional_time||epoch_millis"
      },
      "trackingId" : {
        "type" : "keyword"
      }
    }
  }
}'
```
* Expected response:
```
{"acknowledged":true}
```
* Populate new order with spaces in id and trackingId fields/properties:  
```
curl -XPOST 'localhost:9200/orders/orders/1?pretty=true' \
  -H 'content-type: application/json' \
  -d '
{
  "id": "orderId with spaces", 
  "placedOn": "2016-10-17T13:03:30.830Z",
  "trackingId": "trackingId with spaces"
}'
```  
* Let's run first search:
```
curl 'localhost:9200/ordering/orders/_search?pretty=true&q=id:orderId'
```
* Did you get any results?
* Let's run second search:
```
curl 'localhost:9200/ordering/orders/_search?pretty=true&q=trackingId:trackingId'
```
* Did you get any results?  
* What's the difference in behaviour and why?  
* Adding mapping for multi-field:
```
curl -XPUT localhost:9200/orders/orders/_mapping \
  -H 'Content-Type: application/json' \
  -d '
{
  "orders" : {
    "properties":{  
       "streetName":{  
          "type":"text",
          "fields":{  
             "notparsed":{  
                "type":"keyword"
             }
          }
       }
    }
  }
}'
```
* Re-populate the data:
```
curl -XPOST 'localhost:9200/orders/orders/1?pretty=true' \
  -H 'Content-Type: application/json' \
  -d '
{
  "id": "string with spaces", 
  "placedOn": "2016-10-17T13:03:30.830Z",
  "streetName": "name with spaces"
}'
```
* Let's search for the street name:
```
curl 'localhost:9200/ordering/orders/_search?pretty=true&q=streetName:name'
```
* Let's search for the street name on not-parsed field:
```
curl 'localhost:9200/orders/orders/_search?pretty=true&q=streetName.notparsed:name'
```
* Let's search for the street name on not-parsed field again:
```
curl 'localhost:9200/ordering/orders/_search?pretty=true&q=streetName.notparsed:name%20with%20spaces'
```
* What are results in the search #1, #2, and #3; and what is the reason for these results?
