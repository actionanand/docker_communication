# Communication in Docker - Networks / Requests (section 3)

In many application, you'll need more than one container - for **two main reasons**:

1. It's considered **a good practice** to focus each container on **one main task**(e.g. run a web server, run a database, ...)
2. It's **very hard** to configure a Container that **does more than one ***main thing***** (e.g. run a web server AND a database)

Multi-Container apps are quite common, especially if you're working on **real applications**.

Often, some of these Containers need to **communicate** though:
* either **with each other**
* or with **the host machine**
* or with **the world wide web**

## Communicating with the World Wide Web (WWW)

Communicating with the WWW (i.e. sending Http request or other kinds of requests to other servers) is thankfully very easy.

Consider this JavaScript example - though it'll always work, no matter which technology you're using:

```js
fetch('https://some-api.com/my-data').then(...)
```

This very basic code snippet tries to send a `GET` request to `some-api.com/my-data`.

This will **work out of the box**, no extra configuration is required! The application, running in a Container, will have no problems sending this request.

## Communicating with the Host Machine

Communicating with the Host Machine (e.g. because you have a database running on the Host Machine) is also quite simple, though it **doesn't work without any changes**.

**One important note**: If you deploy a Container onto a server (i.e. another machine), it's very unlikely that you'll need to communicate with that machine. Communicating to the Host Machine typically is a requirement during development - for example because you're running some development database on your machine.

Again, consider this JS example:

```js
fetch('localhost:3000/demo').then(...)
```

This code snippet tries to send a `GET` request to some web server running on the local host machine (i.e. **outside** of the Container but **not** the WWW).

On your local machine, this would work - inside of a Container, it **will fail**. Because `localhost` inside of the Container refers to the Container environment, not to your local host machine **which is running the Container / Docker!**

But Docker has got you covered!

You just need to change this snippet like this:

```js
fetch('host.docker.internal:3000/demo').then(...)
```

`host.docker.internal` is a special address / identifier which is translated to the IP address of the machine hosting the Container by Docker.

**Important**: _"Translated" does **not** mean that Docker goes ahead and changes the source code. Instead, it simply detects the outgoing request and is able to resolve the IP address for that request._

## Communicating with Other Containers

Communicating with other Containers is also quite straightforward. You have two main options:

1. **Manually find out the IP** of the other Container (it may change though)
2. Use **Docker Networks** and put the communicating Containers into the same Network

Option 1 is not great since you need to search for the IP on your own and it might change over time.

Option 2 is perfect though. With Docker, you can create Networks via `docker network create SOME_NAME` and you can then attach multiple Containers to one and the same Network.

Like this:

```shell
docker run -network my-network --name cont1 my-image
docker run -network my-network --name cont2 my-other-image
```

Both `cont1` and `cont2` will be in the **container names** under same Network.

Now, you can simply **use the Container names** to let them communicate with each other - again, Docker will resolve the IP for you (see above).

```js
fetch('cont1/my-data').then(...)
```

## Commands used in this project

Here are the commands used in this project

### Using host machine's DB

1. Connecting mongo (docker) from host machine

Replace `mongodb://localhost:27017` by `mongodb://host.docker.internal:27017`

2. Building the project

```shell
docker build . -t actionanand/docker_communication:tagName
```

3. Running the container

```bash
docker run -d -p 3000:80 --rm --name docker_communication actionanand/docker_communication:tagName
```

### Using docker's DB (with dedicated IP)

1. Running `mongo` image

```shell
docker run -d --rm --name mongoCommunication mongo:7.0
```

2. Finding IP of the mongo (running inside docker)

```shell
docker container inspect mongoCommunication
```

  * Look for the `IPAddress` property under the object `NetworkSettings`
  * Just consider, `172.17.0.3` is the IP, then
  * Replace `mongodb://localhost:27017` by `mongodb://172.17.0.3:27017`

3. Building the project

```shell
docker build . -t actionanand/docker_communication:tagName
```

4. Running the container

```bash
docker run -d -p 3000:80 --rm --name docker_communication actionanand/docker_communication:tagName
```

### Using docker's DB (docker network)

For this, All the containers should be under the same network

1. Create a new docker network

```shell
docker network create network_name
```

```bash
docker network create mongo-net
```

  * You can view the existing networks by `docker network ls`
  * You can remove the unused networks by `docker network rm network_name`
  * `docker network --help` is for help related to docker network

2. Running `mongo` image with volume and docker network

```bash
docker run -d --rm -v mongo-com-vol:/data/db --name mongoCommunication --network mongo-net mongo:7.0
```
  * `-v mongo-com-vol:/data/db` is named volume with the name `mongo-com-vol`
  * You can view the existing volumes by `docker volume ls`
  * You can remove the unused volume by `docker volume rm volume_name`
  * `docker volume --help` is for help related to docker volume

  * Change the DB connection in the project as below:

  ```js
  mongodb://<mongo db container name>:27017
  ```

  ```js
  mongodb://mongoCommunication:27017
  ```

3. Building the project

```shell
docker build . -t actionanand/docker_communication:tagName
```

4. Running the container with docker network

```shell
docker run -d -p 3000:80 --rm --name docker_communication --network mongo-net actionanand/docker_communication:tagName
```

5. You can open the postman app(or any other similar client) and try the following API end points

    1. `GET` request for `http://localhost:3000/movies`
    2. `GET` request for `http://localhost:3000/people`
    3. `POST` request for `http://localhost:3000/favorites` with the following body as example

        ```json
        {
          "name": "A New Hope",
          "type": "movie",
          "url": "https://swapi.dev/api/films/1/"
        }
        ```
        And make sure to keep the `type` as either `movie` or `character`. If you use anything apart from this, error will be thrown from backend
    4. `GET` request for `http://localhost:3000/favorites`

### Bonus

Accessing DB running inside docker from host machine (our local computer with localhost)

1. Exposing mongo(docker) port(27017) to host machine's local host

```bash
docker run -d --rm -p 20243:27017 -v mongo-local-vol:/data/db --name mongoDbLocal mongo:7.0
```
2. Connect to the following URl using [Studio 3T](https://studio3t.com/best-mongodb-gui/) or [MongoDB Compass](https://www.mongodb.com/products/tools/compass)

```js
mongodb://localhost:20243
```

## Docker Image's public URL

* [actionanand/docker_communication](https://hub.docker.com/r/actionanand/docker_communication)

## Associated repos:

1. [Docker Basics](https://github.com/actionanand/docker_playground)
2. [Managing Data and working with volumes](https://github.com/actionanand/docker_data_volume)
3. [Docker Communication](https://github.com/actionanand/docker_communication)
