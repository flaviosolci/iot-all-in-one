# IoT All-in-on

Holds the docker-compose files to run the projects **[iot-consumer-api](https://github.com/flaviosolci/iot-consumer-api)** and **[iot-producer-simulator-api](https://github.com/flaviosolci/iot-producer-simulator-api)**

## Third party images

- Zookeeper: [wurstmeister/zookeeper]( wurstmeister/zookeeper)
- Kafka: [wurstmeister/kafka](wurstmeister/kafka)
- TimescaleDB: [timescale/timescaledb:latest-pg12](timescale/timescaledb:latest-pg12)
- Kafdrop: [obsidiandynamics/kafdrop](obsidiandynamics/kafdrop)

## Getting Started

### Requirements

- Docker

### Running

Below are the options:

1. **A docker-compose to rule them all**:

   1. Install docker: https://docs.docker.com/engine/install/ 

      - On Linux, docker compose need be installed separately: https://docs.docker.com/compose/install/

   2. Run the docker compose file

      ````bash
      docker-compose -f "docker-compose.yml" up -d
      ````

      ![Docker compose success status](docs/wiki/images/docker-compose-all-in-one.png)

      It will start:

      - 1x [Zookeper](https://hub.docker.com/r/wurstmeister/zookeeper)
      - 3x [Kafka](https://hub.docker.com/r/wurstmeister/kafka) brokers
      - [TimescaleDB](https://hub.docker.com/r/timescale/timescaledb) (used by the consumer)
      - [Kafdrop](https://hub.docker.com/r/obsidiandynamics/kafdrop)
      - [Producer API](https://hub.docker.com/layers/flaviosolci/iot-services/producer-latest-develop/images/sha256-f97c1b7ef32b832f1b0dfda637645a7e07c0c7dd3f372007633657f9f15e8afe?context=repo)
      - [Consumer API](https://hub.docker.com/layers/flaviosolci/iot-services/consumer-latest-develop/images/sha256-15a096b68ec8088f7c9e4185fdef254ba081366204856449a84878874fc5e5f4?context=repo)

   4. Consumer will start on `localhost:8081`

   4. Producer will start on `localhost:8080`

      Checkout the [Postman Collection](https://documenter.getpostman.com/view/7376623/T1LLETBq?version=latest#ced80847-a808-4214-8b64-9e3198375944) to know about the endpoint and how execute them.

      :memo: This will also start the consumer API. Checkout more about it: https://github.com/flaviosolci/iot-consumer-api

      :memo: It also comes with [Kafdrop](https://github.com/obsidiandynamics/kafdrop), which is *is a web UI for viewing Kafka topics and browsing consumer groups*. You can access via browser: [http://localhost:9000](http://localhost:9000/).

2. **Hmm, that's ok...**: Cloning the repository + Kafka Docker

   1. Clone this repo to your local machine

   2. Install docker: https://docs.docker.com/engine/install/ 

      - On Linux, docker compose need be installed separately: https://docs.docker.com/compose/install/

   3. Run the docker compose file

      ````bash
      docker-compose -f "docker-compose-minimal.yml" up -d
      ````

      ![Docker compose success status](docs/wiki/images/docker-compose-minimal.png)

      - It will start:
        - 1x [Zookeper](https://hub.docker.com/r/wurstmeister/zookeeper)
        - 3x [Kafka](https://hub.docker.com/r/wurstmeister/kafka) brokers
        - [TimescaleDB](https://hub.docker.com/r/timescale/timescaledb) (used by the consumer)
        - [Kafdrop](https://hub.docker.com/r/obsidiandynamics/kafdrop)

   4. Go to the Producer API and clone the repository: https://github.com/flaviosolci/iot-producer-simulator-api

      1. Go to the root directory of that project
      2. Run `./gradlew bootrun`

   5. Go to the Consumer API and clone the repository: https://github.com/flaviosolci/iot-consumer-api

      1. Go to the root directory of that project
      2. Run `./gradlew bootrun`

   6. Consumer will start on `localhost:80892`

   7. Producer will start on `localhost:8091`

   8. Checkout the [OpenApi](docs/api/openapi.yml) to know about the endpoint and how execute them.

      :memo: It also comes with [Kafdrop](https://github.com/obsidiandynamics/kafdrop), which is *is a web UI for viewing Kafka topics and browsing consumer groups*. You can access via browser: [http://localhost:9000](http://localhost:9000/).


## Documentation

### OpenApi

The endpoints are documented using OpenApi. 

​	For the producer: https://github.com/flaviosolci/iot-producer-simulator-api/blob/master/docs/api/openapi.yml

​	For the consumer: https://github.com/flaviosolci/iot-consumer-api/blob/master/docs/api/openapi.yml

You can also checkout the [Postman Collection](https://documenter.getpostman.com/view/7376623/T1LLETBq?version=latest#ced80847-a808-4214-8b64-9e3198375944).

### Security

The API security is made using JWT authentication. The Authorization Server is provided by [Okta](https://www.okta.com/), so we just need consume an endpoint with the credentials and it will return the token. With this token you can access the API resources.

To access this API, the user must have the `admin` scope, otherwise the access will be denied.

To get the token, you must access the endpoint: `https://dev-928245.okta.com/oauth2/ausp92bbxrQ9m9Ojq4x6/v1/token` and provide: 

- username (email)
- password 
- Authorization Basic

Ex:

```shell
curl --location --request POST 'https://dev-928245.okta.com/oauth2/ausp92bbxrQ9m9Ojq4x6/v1/token' \
--header 'Accept: application/json' \
--header 'Authorization: Basic d0MzR3FzN1RRY0E0dg==...' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'username=test@test.com' \
--data-urlencode 'password=as243fse2' \
--data-urlencode 'grant_type=password'
```

If you have the right permissions it will return you something like that:

```json
{
    "token_type": "Bearer",
    "expires_in": 1800,
    "access_token": "eyJraWQiOiJUNzNuY3dy......",
    "scope": "admin"
}
```

The `access_token` the one you need pass when calling the API

Ex:

```shell script
curl --location --request POST 'http://localhost:8091/producer-api/clusters' \
--header 'Authorization: Bearer eyJraWQiOiJUNzNuY3dy......' \
--header 'Content-Type: application/json' \
--data-raw '[
    {
        "total": 10,
        "type": "CPU",
        "heartBeat": 1,
        "clusterSize": 5,
        "clusterId": 445
    }
]'
```

In case you don't provide the token in the call, an error `401` will be returned. If your user does not have the correct permissions, an error `403`will be returned.

More about JWT: https://auth0.com/learn/json-web-tokens/

##### Authentication flow:

1. Receive the call
2. Check if there is a token in the header
   1. In case negative, thrown an exception (status `401`)
3. Validate the token against Okta public key
   1. If the token is invalid or expired, throw exception (status `401`)
4. Check if the token has necessary scopes to access the resource
   1. If does not have, throw an exception (status `403`)
5. Procced to complete the request
