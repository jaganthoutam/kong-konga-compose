# kongpose

   * Use if you Enables CIS OS Hardening.
```bash
systemctl restart firewalld
systemctl enable firewalld
firewall-cmd --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=443/tcp
firewall-cmd --zone=public --add-port=6443/tcp --permanent
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

After logging in as admin, create a new connection with URL `http://kong:8001`.

Konga uses MongoDB (4.1) with a persistent Docker volume for its credentials.


