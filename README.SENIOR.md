# Context

The goal here is to provide an API that solves our airline inventory problem stated below.
As described in the Key Points sections: **We would be looking at your code the same way (almost) we would be looking at production code.**

## Route

A route is a direct flight between an originating airport (departure) and arrival airport. 

## Code share

Two airlines could come to an agreement where the operating airline (the one operating the flight) would allow an other carrier to sell its own inventory. This agreement is called a code share, and is decided on a per route basis.
For instance, you could have BA (British Airways) operating a flight from Paris to London, and also allowing Air France to sell this same flight. However, other companies cannot sell this flight unless they also come to a code share agreement with British Airways.

## Inventory 

Our inventory consists of records of all routes operated by every airline, along with their code share agreements.
This gets materialized by a table `t_routes`, which looks like:


| id  | operating_airline  | from  | to  | code_shares |
|---|---|---|---|---|
| ab348696-13cf-4aeb-b695-6e7061729fcb  | AF  | CDG  | JFK  | BA,LH  |
| 9196742a-347d-4d87-9e97-e76f42ae9eb8  | AF  | CDG  | NCE  | U2,BA  |
| c3d3daf9-b51f-4ccb-b881-a7fcafc527f3  | AF  | NCE  | JFK  | NULL |


Where:
 * `id` is a uuid for the given row
 * `operating_airline` is the two letters airline code  (e.g `AF`: Air France, `BA`: British Airways, `U2`: Easyjet..)
 * `from` is the originating airport code
 * `to` is the arrival airport code 
 * `code_shares` is a comma separated list of airline codes

Airport codes are three letters  [IATA Codes](https://en.wikipedia.org/wiki/IATA_airport_code) 
Airline codes are two letters [IATA Codes](https://en.wikipedia.org/wiki/List_of_airline_codes) 

If we assume that the previous data table was our whole inventory, then that would mean: 
 * `AF` sells direct flights from  `CDG` to `JFK`
 * `AF` allows both `BA` and `LH` to sell its flight connecting `CDG` to `JFK`
 * `AF` sells direct flights from  `CDG` to `NCE`
 * `AF` allows `U2` and `BA` to sell its flight connecting `CDG` to `NCE`
 * `AF` sells direct flights from  `NCE` to `JFK`
 * The `NCE` to `JFK` flight can **only** be sold by `AF`


# API

As described in the introduction, the goal here is to create an API that given an airline and an airport couple will return possible flight combinations.
This gets materialized by a single endpoint `GET /api/routes/$(airline_code)/$(from)/$(to)`. 

Let's assume our inventory is entirely equal to:

| id  | operating_airline  | from  | to  | code_shares |
|---|---|---|---|---|
| ab348696-13cf-4aeb-b695-6e7061729fcb  | AF  | CDG  | JFK  | BA,LH  |
| 9196742a-347d-4d87-9e97-e76f42ae9eb8  | AF  | CDG  | NCE  | U2,BA  |
| c3d3daf9-b51f-4ccb-b881-a7fcafc527f3  | AF  | NCE  | JFK  | NULL |

A call to `GET /api/routes/AF/CDG/JFK` will return all the available routes allowing a user flying from `CDG` (Paris) to `JFK` (New York). 

In our example dataset, a user willing to fly from `CDG` to `JFK`, could either way: 
 * Fly directly from `CDG` to `JFK`
 * Fly from `CDG` to `JFK` with a stopover at `NCE` (Nice) airport

However a call to `GET /api/routes/BA/CDG/JFK` will look for the same available routes, but this time sold by `BA` or with another company in agreement (codeshare) with `BA`. `BA` does not operate any flight from `CDG` to `JFK` according to our inventory. However, `BA` is allowed to sell the `AF` flight flying from `CDG` to `JFK`, but is not allowed to sell the `CDG` -> `NCE` -> `JFK` as there is no code share for the last part `NCE` -> `JFK`. Therefore the only available options are:
 * Fly directly from `CDG` to `JFK` (sold by `BA` and operated by `AF`)


Please note that a returned combination of routes **cannot** have more than three stopovers


## Response structure

It is ***up to you*** to choose the response structure. The only constraint is to use JSON. 
We only need to have all the available information. For instance, if the API returns a combination with a stopover, then the stopover airport must be cleary stated in the response.

For a call to `GET /api/routes/AF/CDG/JFK`, we would get the following values: 

| operating_airlines  | selling_airline  | from  | to | stopovers |
|---|---|---|---|---|
| AF  | AF  | CDG  | JFK  | | 
| AF  | AF  | CDG  | JFK  | NCE |

For a call to `GET /api/routes/BA/CDG/JFK`, we would get the following values: 

| operating_airlines  | selling_airline  | from  | to | stopovers |
|---|---|---|---|---|
| AF  | BA  | CDG  | JFK  | | 


## More complex scenarios

### Multiple shares

If our inventory was to be:

| id  | operating_airline  | from  | to  | code_shares |
|---|---|---|---|---|
| ab348696-13cf-4aeb-b695-6e7061729fcb  | AF  | CDG  | NCE  | NULL  |
| c3d3daf9-b51f-4ccb-b881-a7fcafc527f3  | BA  | NCE  | JFK  | AF |

Then this would mean that `AF` is operating a `CDG` to `NCE` (without code share), and `BA` is operating a `NCE` to `JFK` with a code share to `AF`. Therefore if one was to call `GET /api/routes/AF/CDG/JFK`, then the following results would be returned:

| operating_airlines  | selling_airline  | from  | to | stopovers |
|---|---|---|---|---|
| AF,BA  | AF  | CDG  | JFK  | NCE| 


### Loop


If our inventory was to be:

| id  | operating_airline  | from  | to  | code_shares |
|---|---|---|---|---|
| ab348696-13cf-4aeb-b695-6e7061729fcb  | AF  | CDG  | JFK  | NULL  |
| c3d3daf9-b51f-4ccb-b881-a7fcafc527f3  | AF  | NCE  | JFK  | NULL |
| c3d3daf9-b51f-4ccb-b881-a7fcafc527f3  | AF  | NCE  | CDG  | NULL |


Then if someone was looking for routes between `CDG` and `JFK` with `AF`, you would need to make sure you don't end up in an infinite loop: `CDG` -> `NCE` -> `CDG` -> `NCE` ..... -> `JFK`

To avoid such problems, the API must not return a combination in which the originating airport is also in the list of stopovers. Moreover, do not forget that the each combination cannot have more than three stopovers.

# Constraint

There is no technical constraint on this test. It's up to you to decide: 
 * The language
 * Whether you build the API using in-memory solutions, using DB....

Really it's up to you to make your own choices. 

The only constraints are:
 * You **must** use the provided datasets (see below). For testing purpose, you can create your own datasets of course.
 * There **cannot** be more than three stopovers in a returned flight.
 
The solution does not have to be highly scalable. It's better if your solution can handle an inventory 1000 times bigger, but this is not mandatory.

 # Datasets

We provide two inventories as CSV files:
 * [routes.csv](dataset/routes.csv)
 * [routes-without-codeshare.csv](dataset/routes-without-codeshare.csv) which is the same file as above but without any code share

# What is expected

We split the exercise in three steps:
 * [Mandatory] We expect you to provide the API in a context where there is no code share. The associated dataset is the one provided in the [routes-without-codeshare.csv](dataset/routes-without-codeshare.csv) file
 * [Bonus #1] If you still have time, we'd like to see a working solution for the case where we have code shares in place. In this scenario, the dataset to use is the one ine the [routes.csv](dataset/routes.csv) file.
 * [Bonus #2] For this one we do not expect any code to be written. This is described below.

# Bonus // No code required

The following points are not required, but provide an extra bonus ;) 
We do not expect you to write the code for them, but rather explain how you would tackle them.


 * We would like to be able to cache and reuse results.
 * We would like the endpoint to be secured. 
 * Once security and identification in place, we need to be able to rate limit this API. 
 

# Key points

The key points we will be looking at are:

 * Tests & testability
 * Architecture and design
 * Code quality
 * Tech choices

We would be looking at your code the same way we would be looking at production code. 
We know you may not have the time to make everything work fine, so it's ok to create dummy functions i.e functions that do nothing but are important for the process. 

If you take any shortcurt in your code, please comment it properly why you did it and how you would do it properly.