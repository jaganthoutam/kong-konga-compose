# kongpose

   * Use if you Enables CIS OS Hardening.
```bash
systemctl restart firewalld
systemctl enable firewalld
firewall-cmd --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=443/tcp
firewall-cmd --zone=public --add-port=6443/tcp --permanent
firewall-cmd --zone=public --add-port=2377/tcp --permanent
firewall-cmd --zone=public --add-port=2379/tcp --permanent
firewall-cmd --zone=public --add-port=2380/tcp --permanent
firewall-cmd --zone=public --add-port=10250/tcp --permanent
firewall-cmd --zone=public --add-port=8000-8001/tcp --permanent
firewall-cmd --zone=public --add-port=8443-8444/tcp --permanent
firewall-cmd --zone=public --add-port=1337/tcp --permanent
firewall-cmd --zone=public --add-port=27017/tcp --permanent
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --permanent --zone=public --change-interface=docker0
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 4 -i docker0 -j ACCEPT
firewall-cmd --reload
systemctl restart firewalld
systemctl restart docker
```


  * Preparation
  ```bash
   yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
   sudo curl  https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
   sudo yum makecache
   sudo dnf -y install docker-ce
   sudo dnf -y install  git
   sudo systemctl enable --now docker
   sudo curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose && ln -sv /usr/local/bin/docker-compose /usr/bin/docker-compose
   sudo docker-compose --version
   sudo docker --version
  ```

Run [Kong API Gateway, Community Edition](https://konghq.com/kong-community-edition)
development setup with [docker-compose](https://docs.docker.com/compose).
Includes [Konga](https://github.com/pantsel/konga) as admin webapp.


## Docker-compose Usage

 ```bash
    git clone https://github.com/jaganthoutam/kong-konga-compose.git
    cd kong-konga-compose
    docker-compose up
  ```
  
## Kong Cluster Swarm Usage(HA)


  ```bash
    
    #In First Node:
    
    # Apply all Preparation Commands 
    
    git clone https://github.com/jaganthoutam/kong-konga-compose.git
    
    cd kong-konga-compose
    
    docker swarm init --advertise-addr MASTERNODEIP
    
    
    OUTPUT:
    
    docker swarm join --token SWMTKN-1-1t1u0xijip6l33wdtt7jpq51blwx0hx3t54088xa4bxjy3yx42-90lf5b4nyyw4stbvcqyrde9sf MASTERNODEIP:2377
    
    
    #In 2nd Node:
    
    # Apply all Preparation Commands 
    
    # The command you find in MASTER NODE.
    
    docker swarm join --token SWMTKN-1-1t1u0xijip6l33wdtt7jpq51blwx0hx3t54088xa4bxjy3yx42-90lf5b4nyyw4stbvcqyrde9sf MASTERNODEIP:2377
    
    #In First Node:
    
    [root@master01 vagrant]#docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
m55wcdrkq0ckmtovuxwsjvgl1 *   master01            Ready               Active              Leader              19.03.8
e9igg0l9tru83ygoys5qcpjv2     node01              Ready               Active                                  19.03.8
    
    
    #Docker Deploy with compose file
    
    docker stack deploy --compose-file=docker-compose-swarm.yaml kong
    
    #Check Services
    
    docker service ls
ID                  NAME                  MODE                REPLICAS            IMAGE                             PORTS
ahucq8qru2xx        kong_kong             replicated          1/1                 kong:1.4.3                        *:8000-8001->8000-8001/tcp, *:8443->8443/tcp
bhf0tdd36isg        kong_kong-database    replicated          1/1                 postgres:9.6.11-alpine
tij6peru7tb8        kong_kong-migration   replicated          0/1                 kong:1.4.3
n0gaj0l6jyac        kong_konga            replicated          1/1                 pantsel/konga:latest              *:1337->1337/tcp
83q1eybkhvvy        kong_konga-database   replicated          1/1                 mongo:4.1.5                       *:27017->27017/tcp 
    
  ```


## Endpoints

### Kong

- Proxy: [http://localhost:8000](http://localhost:8000)
- Proxy w/ SSL: [https://localhost:8443](https://localhost:8443)
- Admin API: [http://localhost:8001](http://localhost:8001)

Kong uses PostgreSQL (9.6) with a persistent Docker volume for its credentials.

### Konga

- GUI: [http://localhost:1337](http://localhost:1337)

The following default users are configured in `konga/user_seed.js`:
- admin / adminadminadmin
- demo / demodemodemo

Konga uses MongoDB (4.1) with a persistent Docker volume for its credentials.



  * Add Kong Admin API as services:
  ```bash
  curl --location --request POST 'http://localhost:8001/services/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "admin-api",
    "host": "localhost",
    "port": 8001
}'
  OUTPUT:
  {"host":"localhost","created_at":1589537495,"connect_timeout":60000,"id":"ba833b38-f22f-44bc-9173-d4d78d45ca50","protocol":"http","name":"admin-api","read_timeout":60000,"port":8001,"path":null,"updated_at":1589537495,"retries":5,"write_timeout":60000,"tags":null,"client_certificate":null}
  ```

Add Admin API route: To register route on Admin API Services we need either service name or service id, you can replace the following command below:
```bash
curl --location --request POST 'http://localhost:8001/services/admin-api/routes' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paths": ["/admin-api"]
}'

OUTPUT:

{"id":"7e65e0b9-3329-4cfb-beb4-20276b0b96ee","tags":null,"updated_at":1589537543,"destinations":null,"headers":null,"protocols":["http","https"],"created_at":1589537543,"snis":null,"service":{"id":"ba833b38-f22f-44bc-9173-d4d78d45ca50"},"name":null,"preserve_host":false,"regex_priority":0,"strip_path":true,"sources":null,"paths":["\/admin-api"],"https_redirect_status_code":426,"hosts":null,"methods":null}

```

Now our Kong Admin API is running behind Kong Proxy, so in order to access it you need to :
```bash
curl localhost:8000/admin-api/
```

Enable Key Auth Plugin
Our Admin API already run behind kong, but is not secured yet. In order to protect Kong Admin API we need to enable key auth plugin at service level by doing this commands.
```bash
curl -X POST http://localhost:8001/services/admin-api/plugins \
    --data "name=key-auth" 

OUTPUT:
{"created_at":1589537817,"config":{"key_names":["apikey"],"run_on_preflight":true,"anonymous":null,"hide_credentials":false,"key_in_body":false},"id":"a1ef37a3-9724-4077-b2c9-f28b2b47b70b","service":{"id":"ba833b38-f22f-44bc-9173-d4d78d45ca50"},"name":"key-auth","protocols":["grpc","grpcs","http","https"],"enabled":true,"run_on":"first","consumer":null,"route":null,"tags":null}

```

Add Konga as Consumer
Our Admin API already run behind kong, but is not secured yet. In order to protect Kong Admin API we need to enable key auth plugin at service level by doing this commands.
```bash
curl --location --request POST 'http://localhost:8001/consumers/' \
--form 'username=konga' \
--form 'custom_id=cebd360d-3de6-4f8f-81b2-31575fe9846a'

OUTPUT:
{"custom_id":"cebd360d-3de6-4f8f-81b2-31575fe9846a","created_at":1589537912,"id":"8866a8a4-722d-4da5-bf6a-b44c72abbb70","tags":null,"username":"konga"}
```

Create API Key for Konga
Using Consumer ID that generated when we're adding consumer, we will use that Consumer ID and generate API Key.  (Previous Step we got  consumer ID : 8866a8a4-722d-4da5-bf6a-b44c72abbb70)
```bash
curl --location --request POST 'http://localhost:8001/consumers/8866a8a4-722d-4da5-bf6a-b44c72abbb70/key-auth'

OUTPUT : 

{"created_at":1589537982,"consumer":{"id":"8866a8a4-722d-4da5-bf6a-b44c72abbb70"},"id":"b214c70c-4290-44e9-b283-ee5f05437349","tags":null,"ttl":null,"key":"4RB6Y5KPrW5nl0DQqx1da2n9YJdJfl"}

```

Setup Connection
Now we're already have all required component to setup Konga connection

![Image Konga Connections](https://github.com/jaganthoutam/kong-konga-compose/blob/master/images/screen2.png)


Add New Connection with Auth Key
  ```bash
  name: admin-api
  API URL: http://kong:8000/admin-api
  API KEY: 4RB6Y5KPrW5nl0DQqx1da2n9YJdJfl
  ```
![Image Konga Connections](https://github.com/jaganthoutam/kong-konga-compose/blob/master/images/screen3.png)


Activate Connection

![Image Activate Connections](https://github.com/jaganthoutam/kong-konga-compose/blob/master/images/screen5.png)



##Finally remove 8801 port from docker-compose file
- Admin API PORT [docker-compose.yaml](https://github.com/jaganthoutam/kong-konga-compose/blob/master/docker-compose.yml#L52) from docker-compose file
```bash
....
....
....
    ports:
      - 8000:8000
      - 8001:8001
   #   - 8443:8443
....
....
....
```
Update docker-compose 
```bash
docker-compose up -d
```




