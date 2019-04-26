# Objective

The goal here is to create an API that returns flights information. Basically we're trying to develop an API that supplies aggregated flight searches results across multiple providers (skyscanner clone). 
This gets materialized by a single endpoint `GET /api/flights`. In order to ease the test, this endpoint does not take any parameter.

As described in the Key Points sections: **We would be looking at your code the same way we would be looking at production code.**

# Context

The inventory is maintained by three providers:
 * Air-Jazz
 * Air-Moon
 * Air-Beam

All these suppliers provide their inventory through a simple API. These API are provided using [Mockaroo](https://mockaroo.com), and therefore all the data are randomly generated on every call.

## Air Jazz

The inventory can be retrieved by an HTTP call :

```shell
curl -H "X-API-Key: dd764f40" https://my.api.mockaroo.com/air-jazz/flights
```

This API returns JSON formatted flights information. Below is a sample:

```javascript
[
  {
    "id": "c2e91bdf-ccc0-45d5-b4ea-ef75bc932ae8",
    "price": 511.78,
    "dtime": "9:15 PM",
    "atime": "5:35 AM"
  },...
]
```

Where: 
 * `id` is a unique identifier
 * `price` is the actual price for this flight 
 * `dtime` is the departure time for this flight
 * `atime` is the arrival time


 
## Air Moon
 
 The inventory can be retrieved by an HTTP call :
 
 ```shell
 curl -H "X-API-Key: dd764f40" https://my.api.mockaroo.com/air-moon/flights
 ```
 
 This API returns JSON formatted flights information. Below is a sample:
 
 ```javascript
 [
  {
    "id": "e597f52b-02ad-40f5-8810-8aa7d8d8769c",
    "price": 486.88,
    "departure_time": "7:02 AM",
    "arrival_time": "2:50 AM"
  },...
 ]
 ```
 
 Where: 
  * `id` is a unique identifier
  * `price` is the actual price for this flight 
  * `departure_time` is the departure time for this flight
  * `arrival_time` is the arrival time

## Air Beam
  
The inventory can be retrieved by an HTTP call :

```shell
curl -H "X-API-Key: dd764f40" https://my.api.mockaroo.com/air-beam/flights
```

This API returns CVS formatted flights information. Below is a sample (firs three lines):

```
id,p,departure,arrival
14e6f085-b5b5-48f7-b3c5-6c6202d50f48,501.33,4:12 AM,5:02 AM
46ea7e60-c0a4-429a-8917-3917d903236d,497.0,7:22 PM,5:58 AM  
```

Where: 
 * `id` column is a unique identifier
 * `p` column is the actual price for this flight 
 * `departure_time` column is the departure time for this flight
 * `arrival_time` column is the arrival time

# Features

As described above, we aim to provide a single endpoint that will aggregate results from these three suppliers. Whenever one sends a `GET /api/flights` on our API, then the program should retrieve all results from the suppliers, sort them accordingly to their price (ascending), and limit the number of resuls to 50 flights.

Our API should return a json array containing the following schema:

```
[
  {
    "provider": "AIR_MOON|AIR_JAZZ|AIR_BEAM", // one of the supplier
    "price": <double>,
    "departure_time": <time>,
    "arrival_time": <time>
  }
]
```

Beyong these *basic* features, we will be interested in finding out how you would handle the following use cases / scenarios:
 * Provider `Air Moon` frequently takes a long time to respond (but it does send back data). Depending on the way you developed the API it may have performance impacts on the whole search. How would you take care of this ? 
 * Provider `Air Jazz` has downtime issues from time to time, and returns a `HTTP 502 Bad Gateway` error. Once again, how would you handle this so it does not penalize the whole API.
 * The API we just created is to be used by our partners. How would you handle security ? We need to make sure only authenticated users (and authorized) can access this API.
 * We would want to rate limit our API, so each of our client has a limited number of allowed calls. How would you handle this ?
 * Imagine we now have a lot of incoming traffic on our API, and there is some overlap on the search requests. How could we improve the program ?
 * Anything that you think could be relevant....

 

# Key points

The key points we will be looking at are:

 * Architecture and design
 * Code quality
 * Tests & testability
 * Tech choices

We know you may not have the time to make everything work fine, so it's ok to create dummy functions i.e functions that do nothing but are important for the process. 