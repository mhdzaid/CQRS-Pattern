## Project Structure
* The project consists of the following microservices.
  * [Gateway](https://github.com/mhdzaid/gateway)
  * [Server](https://github.com/mhdzaid/server)
  * [User](https://github.com/mhdzaid/User)
  * [Location-Writer](https://github.com/mhdzaid/location-writer)
  * [Location-Reader](https://github.com/mhdzaid/location-reader) 
* `How To Run` these projects is given in their respective `README.md`  
* `Eureka` is used for microservice discovery along with `Spring Cloud` for loadbalancing.
* The service [Location-Writer](https://github.com/mhdzaid/location-writer) is used to recieve location data. Since there will be a lot of request coming to this end point I've created multiple instances of this service which are loadbalanced.
*  The loadbalancing startegy is the default `Round Robbin`, I've also created a custom strategy based on `URL hash` for server stickiness. The reason for this was because we have multiple instances of `Location` microservice, the data of a particular user would be distributed, making pagination and data retrieval complex.
*  However my custom `URL hash` load balancer depends on number of instances for server stickiness, so if instances increases the data would be distributed again.
*  Keeping this in mind I created another microservice [Location-Reader](https://github.com/mhdzaid/location-reader) which would sync with the [Location-Writer](https://github.com/mhdzaid/location-writer).
*  The [Location-Reader](https://github.com/mhdzaid/location-reader)'s database would have partitions as well as indexes for fast retrival.
*  These two databases sync with each other by an Async method that send location data through kafka from [Location-Writer](https://github.com/mhdzaid/location-writer) to [Location-Reader](https://github.com/mhdzaid/location-reader). (Previously I used [Debezium](https://debezium.io/) but that seemed over kill, as well as a few problems I faced with Debezium connector).
* In [Location-Reader](https://github.com/mhdzaid/location-reader) Database `Location` is partitioned based on `user_id` column for fast retrieval since a user can have thousands of entries.
* `created_on` column has been indexed as well for doing time frame query.
* The `id` column is an auto-increment column so the latest location of the user is the last value inserted in the partition.
