## Task 1.

### The gay border TOIlet

The TOIlet (stands for “The Other Implementation’s letters”) application is a linux CLI that takes the input characters and produces various text-based effects. I want a container that can output me colorful variations of a given text.

See [TOIlet's website](http://caca.zoy.org/wiki/toilet) for more info (but you do not have to).

**Tasks:**

Write a build file that:

* takes an ubuntu 18.04
* instals toilet.
* makes sure that the commands `toilet -F border --gay` always will be run before any user inputted commands to the container.

Submit your build file.

Build your image, calling it `<yourusername>/toilet`.

Push your newly created image up to dockerhub, and provide the link to it.

**Solution**

1. Make a new directory to contain the Docker image files
2. Make a Docker file containing the following:
```bash
# Install the image from docker hub
FROM ubuntu:18.04
# Update the ubuntu distro
RUN apt-get update -y && \
apt-get install toilet -y
# Run the toilet command when the container starts
# as the default command. This will always be executed before user input.
ENTRYPOINT ["toilet", "-F", "border", "--gay"]
CMD  ["DEFAULT MESSAGE"]
```

3. Run the command
	`docker build -t ubuntu-toilet:latest .` to build the image
4. Run the command
	`docker run ubuntu-toilet:latest hello world` to run the container and write hello world to the std output
6. Upload the image to docker hub by first tagging the image using the following command
	 `docker tag ubuntu-toilet:latest csbc92/toilet:latest`
7. and then 
	`docker push csbc92/toilet:latest` 

8. Link to image https://hub.docker.com/r/csbc92/toilet/


## Task 2.

We want to run a wordpress site, that sits behind a proxy server. You do not need any experience with proxies, nor Nginx in particular to solve this assignment.

You need to provide a docker compose file with the following containers in:

- [x] Nginx as a proxy (in the script called loadbalancer-nginx)
- [x] Mysql as a database (in the script called db)
- [x] Wordpress as an application (in the script called wordpress)

Tasks:

- [x] Make nginx the only container visible to the outside world, and only on port 80.

This is done by not exposing the ports by NOT using the "ports:" construct in the docker-compose.yml file.

- [x] Make the containers start in the following order: mysql,wordpress,nginx

This is done by the tag `depends_on:` in the `docker-compose.yml` for each of the containers wordpress and loadbalancer-nginx

```bash
version: '3.1'
services:
  # The wordpress container
  wordpress:
  # .....
    depends_on:
      - db
  # .....
  db:
  # .. no need for depends_on tag on the db..

  loadbalancer-nginx:
  # .....
    depends_on:
      - db
      - wordpress
  # .....
```

- [x] Make nginx volume in the file `nginx.conf` on the container path `/etc/nginx/conf.d/default.conf`

This is done by the following in the `docker-compose.yml`:
```
volumes:
	  - ./nginx.conf:/etc/nginx/conf.d/default.conf`
```

- [x] Make wordpress and mysql configured so they do not need to ask for database host, database name, user and password (hint: look at their docker-hub pages)

This is done by the following in the `docker-compose.yml` (the username root is default in mysql):
```bash
wordpress:
   environment:
	  WORDPRESS_DB_USER: root
	  WORDPRESS_DB_PASSWORD: mysecret
	  WORDPRESS_DB_NAME: wordpress

db:
   environment:
	  MYSQL_ROOT_PASSWORD: mysecret
```

[*] Make a network that all containers belong to.
This is done by the following in the `docker-compose.yml`:
```bash
networks:
# define the networks
  my-basic-network:
```

Describe what kind of commands you would use to delete the containers and create new ones.

```bash
docker-compose down # will stop and remove containers and networks created by the docker compose file. Anonymous Volumes will not by default be removed but can be removed by adding the -v switch. See documentation here: https://docs.docker.com/compose/reference/rm/
docker-compose up -d # will create new containers for the application
```

Describe where you would define what exact version of mysql docker should use?

This can be done in the `docker-compose.yml` file under the image tag for mysql

```bash
db:
  # The mysql container
	image: mysql:5.7 # <---------- mysql version
```

What commands will give you the ip addresses of the containers in the described network.

1. First run the command `docker network ls` to get the list of networks and the ID of the newly created network
2. Run the command `docker network inspect <network_id>` to get the output of the network status and configuration
The output in my case looks like the following, where the tag `"Containers"` will list the ID of each container along with its internal IP address in the subnet.

```
[
	{
		"Name": "wordpresswithproxy_my-basic-network",
		"Id": "a6f8c4ff9390afe7be624c5cd1f05b519bef2f64c1d32b40d07137c10514856e",
		"Created": "2018-08-20T18:10:55.334561892Z",
		"Scope": "local",
		"Driver": "bridge",
		"EnableIPv6": false,
		"IPAM": {
			"Driver": "default",
			"Options": null,
			"Config": [
				{
					"Subnet": "172.19.0.0/16",
					"Gateway": "172.19.0.1"
				}
			]
		},
		"Internal": false,
		"Attachable": true,
		"Ingress": false,
		"ConfigFrom": {
			"Network": ""
		},
		"ConfigOnly": false,
		"Containers": {
            "5c2287e5093b063ad5a109b7e239f4ad667096cf7a9f2eb26edeb5ece5a0f7cd": {
                "Name": "wordpresswithproxy_wordpress_1",
                "EndpointID": "3dd386d7718c7e00fdcb292cb3222cade980da9022eb24e72f44a98042a3ea08",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "6cc8b5a8058fe44432a1368cb3241d0b38f10a0fcb180ba808a0dcff9b7b26a5": {
                "Name": "wordpresswithproxy_loadbalancer-nginx_1",
                "EndpointID": "2ad4691c9e6616faee531083fc3ad38fd2752c8b8209fea3226b62727518f7b4",
                "MacAddress": "02:42:ac:13:00:04",
                "IPv4Address": "172.19.0.4/16",
                "IPv6Address": ""
            },
            "b6b296033814968dd6d7f4332e0f50b1f25f2f3b1db0e43a0227f21a2e18578f": {
                "Name": "wordpresswithproxy_db_1",
                "EndpointID": "d4bc8a860e01983bfe660ff23df717ed9b37eb4118a35457bcf3b4a469aa7352",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
		"Options": {},
		"Labels": {
			"com.docker.compose.network": "my-basic-network",
			"com.docker.compose.project": "wordpresswithproxy"
		}
	}
]
```


Approach:
1. Create a new directory for the docker containers e.g. mkdir wordpress-with-proxy
2. Create a new file called docker-compose.yml with the following content:

```bash
version: '3.1'
services:
  # The wordpress container
  wordpress:
	image: wordpress:latest
	restart: always
	environment:
	  WORDPRESS_DB_USER: root
	  WORDPRESS_DB_PASSWORD: mysecret
	  WORDPRESS_DB_NAME: wordpress
	networks:
	  - my-basic-network
	links:
	  - db:mysql
	depends_on:
	  - db

  db:
  # The mysql container
	image: mysql:5.7
	command: --default-authentication-plugin=mysql_native_password
	restart: always
	environment:
	  MYSQL_ROOT_PASSWORD: mysecret
	networks:
	  - my-basic-network


  loadbalancer-nginx:
  # define the nginx-container
	image: nginx
	volumes:
	  - ./nginx.conf:/etc/nginx/conf.d/default.conf
	ports:
	  - 80:80
	networks:
	  - my-basic-network
	depends_on:
	  - db
	  - wordpress

networks:
# define the networks
  my-basic-network:
```

3. Start the containers by issuing the command
```bash
docker-compose up -d  # d for detached mode
```

4. If there is any problems, one can use the command `docker-compose logs [<container-id>]` to see problems. This helped in a situation where I forgot to add the link between the wordpress container and db container. This caused a 502 Bad Gateway when i connected to the nginx proxy server. The cause was that there was no connection from nginx to the wordpress container, which made the wordpress container restart every 30 seconds.
5. Verify that all containers are running by issuing the command
```bash
docker container ls
```
7. Verify that the correct `nginx.conf` file is actually mapped to the path `/etc/nginx/conf.d/default.conf` by issuing command and verify the content:
```bash
docker container exec -it wordpresswithproxy_loadbalancer-nginx_1 cat /etc/nginx/conf.d/default.conf
```