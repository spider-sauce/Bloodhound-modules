# AFtheM - MongoDB module

The module implements a number of functions requiring interaction with the MongoDB database.

## Configuration Loader

With this loader you can store and load configuration items in MongoDB.

**class:** `com.apifortress.afthem.modules.mongodb.config.loaders.MongoDbConfigLoader`

**configuration:**

Edit the `afthem.yml` file and update the `config_loader.class` string with the provided class.
Add a `params` map to the `config_loader`map to set the connection parameters.
Example:

```yaml
config_loader:
  class: com.apifortress.afthem.modules.mongodb.config.loaders.MongoDbConfigLoader
  params:
    database: afthem
    collection: configuration
    uri: mongodb://localhost
```

Here are some examples of configuration documents

#### backend
```json
{
  "type" : "backend",
  "flow_id" : "default",
  "prefix" : "127.0.0.1/demo",
  "upstream" : "http://demoapi.apifortress.com/api/retail"
}
```

#### flow
```json
{
  "id" : "default",
  "type" : "flow",
  "flow" : {
    "proxy/request" : {
      "next" : "proxy/upstream_http",
      "config" : {},
      "sidecars" : []
    },
    "proxy/upstream_http" : {
      "next" : "proxy/send_back",
      "config" : {},
      "sidecars" : []
    }
  }
}
```

#### implementers
```json
{
  "type" : "implementers",
  "implementers" : [
    {
      "id" : "request",
      "class" : "com.apifortress.afthem.actors.proxy.RequestActor",
      "type" : "proxy"
    },
    {
      "id" : "upstream_http",
      "class" : "com.apifortress.afthem.actors.proxy.UpstreamHttpActor",
      "type" : "proxy"
    },
    {
      "id" : "send_back",
      "class" : "com.apifortress.afthem.actors.proxy.SendBackActor",
      "type" : "proxy"
    }
  ],
  "thread_pools" : {
    "default" : {
      "min" : 1,
      "max" : 1,
      "factor" : 1
    }
  }
}
```

---

## Filters

### MongoApiKeyFilterActor

Filters out any request that does not carry a valid API key in the headers or in the query string.
This base actor loads the API keys from a YAML file.


**class:** `com.apifortress.afthem.modules.mongodb.actors.filters.MongoApiKeyFilterActor`

**sidecars**: yes

**config:**
* `uri`: the MongoDB URI
* `database`: the name of the MongoDB database
* `collection`: the name of the connection to be used
* `in`: either `query` (expecting the key in the query string) or `header` (expecting the key in the headers)
* `name`: key of the field carrying the API key


**Example document**
```json
{
    "api_key" : "123",
    "app_id" : "TheFoobar",
    "enabled" : true
}
```

---
## Sidecars

### MongoSerializerActor

Serializes an API conversation and sends it to MongoDB for storage.

**class:** `com.apifortress.afthem.modules.mongodb.actors.sidecars.serializers.MongoSerializerActor`

**configuration:**

General serializer settings:

* `disable_on_header`: if the provided header is present in the request, then the conversation will skip serialization
* `enable_on_header`: if the provided header is present in the request, then the conversation will be serialized
* `discard_request_headers`: list of request headers that should not appear in the serialized conversation
* `discard_response_headers`: list of response headers that should not appear in the serialized conversation
* `allow_content_types`: full or partial response content types which make the request eligible for serialization. If
the list is null or empty, all content types will be accepted

MongoDB settings:

* `uri`: the MongoDB URI
* `database`: the name of the MongoDB database
* `collection`: the name of the connection to be used