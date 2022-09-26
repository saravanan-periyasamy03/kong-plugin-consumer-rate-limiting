# Consumer Rate Limiting - Kong plugin
---


## Getting Started
Consumer rate limiting plugin extends the capability of built-in Kong [Rate Limiting plugin](https://docs.konghq.com/hub/kong-inc/rate-limiting) and it helps configure consumer based rate limiting for all the API subscriptions at route or service level.

The below table depicts an example configuration of this plugin.

| API/Traffic count   | Consumer 1 | Consumer 2 | Consumer 3 |
|--------------|------------|------------|------------|
| Route/Service 1        | 200        | 50         | 800       |
| Route/Service 2      | 10        | 70         | 400        |
| Route/Service 3        | 150         | 30         | 1500        |



Kong Rate Limiting Plugin offers to apply rate limit on consumers irrespective of their API subscriptions. If the API Provider wants to enforce rate limit for all their resources in a given period of seconds, minutes, hours, days, months, or years and apply it at consumer level then this plugin offers that capability. 

This plugin also goes handy when organization wants to implement monetization for their APIs. Consumer Rate Limiting plugin prevents the DDoS attack at consumer level and on the other side Kong Rate Limiting plugin applied to the route or service will prevent DDoS attack at the API/Resource level. With the Consumer Rate Limiting plugin, genuiene consumers will not be impacted and they will get what rate limit is alloted for them.

If the underlying Service/Route has no authentication layer, the Client IP address will be used; otherwise, the Consumer will be used if an authentication plugin has been configured.

---

## Configuration Reference
This plugin is partially compatible with DB-less mode.

The plugin will run fine with the local policy (which doesn’t use the database) or the redis policy (which uses an independent Redis, so it is compatible with DB-less).

The plugin will not work with the cluster policy, which requires writes to the database.



### Enable the plugin on a consumer at service level

Below options provide configuration to enable consumer rate limiting for a consumer at the service level.
#### Admin API
```
curl -X  POST 'http://localhost:8001/consumers/{CONSUMER_NAME|CONSUMER_ID}/plugins' \
--header 'Content-Type: application/json' \
--data-raw '{
 "name": "consumer-rate-limiting",
 "config":{
    "fault_tolerant": true,
    "policy": "local",
    "service": [
        {
            "name": "{SERVICE_NAME}",
            "second": 500,
            "minute": 10000
        },
        {
            "name": "{SERVICE_NAME}",
            "second":1000,
            "minute": 50000
        }
    ]
}
}'
```
Replace {CONSUMER_NAME|CONSUMER_ID} with the id or name of the consumer that this plugin configuration will target.
Replace {SERVICE_NAME} with the name of the service for which the rate limiting to be applied for an consumer.


### Enable the plugin on a consumer at route level
---
Below options provide configuration to enable consumer rate limiting for a consumerat route level.
#### Admin API
---

    curl -X  POST 'http://localhost:8001/consumers/{CONSUMER_NAME|CONSUMER_ID}/plugins' \
    --header 'Content-Type: application/json' \
    --data-raw '{
     "name": "consumer-rate-limiting",
     "config":{
        "fault_tolerant": true,
        "policy": "local",
        "route": [
            {
                "name": "{ROUTE_NAME}",
                "second": 100,
                "minute": 1000
            },
            {
                "name": "{ROUTE_NAME}",
                "second":500,
                "minute": 5000
            }
        ]
    }
    }'

Replace {CONSUMER_NAME|CONSUMER_ID} with the id or name of the consumer that this plugin configuration will target.
Replace {ROUTE_NAME} with the name of the route for which the rate limiting to be applied for an consumer.


## Parameters
---
Here's a list of all the parameters which can be used in this plugin's configuration:

| FORM PARAMETER	     														| DESCRIPTION										  													|
| ----------- 																		| -----------																								|
| name<br><br>Type:string  														|  The name of the plugin to use, in this case consumer-rate-limiting |
| consumer.id or consumer.name <br><br>Type:string  										  |  The name or ID of the consumer the plugin targets.<br>Not required if using /consumers/CONSUMER_NAME or CONSUMER_ID/plugins |
| enabled<br><br>Type:boolean<br><br>Default value:true   |  Whether this plugin will be applied.										  |
| config.policy<br>*optional*<br><br>Type:string<br><br>Default value: local         |  The rate-limiting policies to use for retrieving and incrementing the limits. Available values are:<br> - local: Counters are stored locally in-memory on the node. - redis: Counters are stored on a Redis server and shared across the nodes.|
|config.fault_tolerant<br>*required*<br><br>Type:boolean<br><br>Default value:true    | A boolean value that determines if the requests should be proxied even if Kong has troubles connecting a third-party data store. If true, requests will be proxied anyway, effectively disabling the rate-limiting function until the data store is working again. If false, then the clients will see 500 errors. |
|config.hide_client_headers<br>*required*<br><br>Type:boolean<br><br>Default value:false | Optionally hide informative response headers.|
|config.redis_host<br>*semi-optional*<br><br>Type:string | When using the redis policy, this property specifies the address to the Redis server.|
|config.redis_port<br>*optional*<br><br>Type:integer<br><br>Default value: 6379  | When using the redis policy, this property specifies the port of the Redis server. By default is 6379.|
|config.redis_username<br>*optional*<br><br>Type:string | When using the redis policy, this property specifies the username to connect to the Redis server when ACL authentication is desired. Added support for redis_username configuration parameter from Kong gateway 2.8.X version|
|config.redis_password<br>*optional*<br><br>Type:string | When using the redis policy, this property specifies the password to connect to the Redis server.|
|config.redis_ssl<br>*required*<br>Type:boolean<br><br>Default value: false | When using the redis policy, this property specifies if SSL is used to connect to the Redis server.|
|config.redis_ssl_verify<br>*required*<br><br>Type:boolean<br><br>Default value: false | When using the redis policy with redis_ssl set to true, this property specifies it server SSL certificate is validated. Note that you need to configure the lua_ssl_trusted_certificate to specify the CA (or server) certificate used by your Redis server. You may also need to configure lua_ssl_verify_depth accordingly.|
|config.redis_server_name<br>*optional*<br><br>Type:string | When using the redis policy with redis_ssl set to true, this property specifies the server name for the TLS extension Server Name Indication (SNI)|
|config.redis_timeout<br>*optional*<br><br>Type:number<br><br>Default value: 2000 | When using the redis policy, this property specifies the timeout in milliseconds of any command submitted to the Redis server.|
|config.redis_database<br>*optional*<br><br>Type:integer<br>Default value: 0 | When using the redis policy, this property specifies the Redis database to use. |
|config.service<br>*optional*<br><br>Type:array of objects | Configure the list of services for which rate limit to be applied for the given consumer |
|config.service.name<br>*required*<br><br>Type:string | Configure the name of the service for which rate limit to be applied for the given consumer |
|config.service.second<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per second for the service defined in the [name] |
|config.service.minute<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per minute for the service defined in the [name] |
|config.service.hour<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per hour for the service defined in the [name] |
|config.service.day<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per day for the service defined in the [name] |
|config.service.month<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per month for the service defined in the [name] |
|config.service.year<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per year for the service defined in the [name] |
|config.route<br>*optional*<br><br>Type:array of objects | Configure the list of routes for which rate limit to be applied for the given consumer |
|config.route.name<br>*required*<br><br>Type:string | Configure the name of the route for which rate limit to be applied for the given consumer |
|config.route.second<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per second for the route defined in the [name] |
|config.route.minute<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per minute for the route defined in the [name] |
|config.route.hour<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per hour for the route defined in the [name] |
|config.route.day<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per day for the route defined in the [name] |
|config.route.month<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per month for the route defined in the [name] |
|config.route.year<br>*optional*<br><br>Type:number | Configure the number of HTTP requests that can be made per year for the route defined in the [name] |

 `Added the redis_ssl, redis_ssl_verify, and redis_server_name configuration parameters from Kong 2.7.X version`

 `Note: At least one limit (second, minute, hour, day, month, year) must be configured. Multiple limits can be configured.`
 
## Headers sent to the client
When this plugin is enabled, Kong sends additional headers to show the allowed limits, number of available requests, and the time remaining (in seconds) until the quota is reset for the consumer at route or service level. Here’s an example header:
```
Consumer-RateLimit-Limit: 50
Consumer-RateLimit-Remaining: 100
Consumer-RateLimit-Reset: 30
```

The plugin also sends headers to show the time limit and the minutes still available:
```
X-Consumer-RateLimit-Limit-Minute: 15
X-Consumer-RateLimit-Remaining-Minute: 10
```
If more than one time limit is set, the header contains all of these:
```
X-Consumer-RateLimit-Limit-Second: 20
X-Consumer-RateLimit-Remaining-Second: 10
X-Consumer-RateLimit-Limit-Minute: 30
X-Consumer-RateLimit-Remaining-Minute: 20
```
When a limit is reached, the plugin returns an HTTP/1.1 429 status code, with the following JSON body:
```
{ "message": "API rate limit exceeded" }
```

`Note: Enterprise-Only: The Kong Community Edition of this Rate Limiting plugin does not include Redis Sentinel support. Only Kong Gateway Enterprise customers can use Redis Sentinel with Kong Rate Limiting, enabling them to deliver highly available primary-replica deployments.`


## Tested in Kong Version
---
Kong OS Gateway 2.8.1

## Steps to use this plugin
---
#### Create a service
```sh
curl -i -X POST \
 --url http://localhost:8001/services/ \
 --data 'name=httpbin-service' \
 --data 'url=http://httpbin.org/anything'
 ```
#### Create a route
```sh
 curl -i -X POST \
 --url http://localhost:8001/services/httpbin-service/routes \
 --data 'hosts[]=example.com' \
 --data 'strip_path=false'
 ```
#### Add Key Auth Plugin to the service
```sh
curl -X POST http://localhost:8001/services/httpbin-service/plugins \
 --data "name=key-auth"  \
 --data "config.key_names=apikey" \
 --data "config.key_in_query=true" 
```
#### Create consumer
```sh
curl -d "username={USER_NAME}&custom_id={CUSTOM_ID}" http://localhost:8001/consumers/
```
#### Generate key for the consumer
```sh
curl -X POST http://localhost:8001/consumers/{USERNAME_OR_ID}/key-auth
```
`Note: Above curl for consumer and key creation works against kong with database mode. For DB-less refer kong documentation for consumer and key generation`

#### Create consumer rate limiting plugin and add it to a consumer
```sh
curl -X  POST 'http://localhost:8001/consumers/{CONSUMER_ID|CONSUMER_NAME}/plugins' \
--header 'Content-Type: application/json' \
--data-raw '{
 "name": "consumer-rate-limiting",
 "config":{
    "fault_tolerant": true,
    "policy": "local",
    "service": [
        {
            "name": "httpbin-service",
            "second": 10,
            "hour": 1000
        },
        {
            "name": "test-service",
            "second": 5,
            "minute": 500
        }
    ]
  }
}'
```

#### Test the Httpbin API
```sh
curl  -H "Host: example.com" 'http://localhost:8000/?apikey={CONSUMER_API_KEY}' -i
```

This plugin will set following response headers which holds the rate limit metrics of the consumer,
```sh
Consumer-RateLimit-Limit: 300
X-Consumer-RateLimit-Limit-Minute: 300
Consumer-RateLimit-Remaining: 298
Consumer-RateLimit-Reset: 49
X-Consumer-RateLimit-Remaining-Minute: 298
```

## Contributers
---
Name | Email Id
--- | --- | 
Anup Kumar Rai | raianup.2407@gmail.com
Saravanan Periyasamy | saravanancse03@gmail.com  
