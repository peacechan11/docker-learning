# Docker Swarm Notes မြန်မာလို

## အကျဉ်းချုပ်

Docker Swarm ဆိုတာ **Docker ရဲ့ native clustering and orchestration tool** ပါ။  
Docker hosts/server အများကြီးကို တစ်စုတစ်စည်းတည်း **cluster** အနေနဲ့ပေါင်းပြီး containers တွေကို run, scale, update, recover လုပ်ပေးနိုင်ပါတယ်။

```text
Docker Swarm = Docker containers တွေကို server အများကြီးပေါ်မှာ cluster အနေနဲ့ manage လုပ်ပေးတဲ့ tool
```

---

# 1. Docker Swarm ဆိုတာဘာလဲ?

Docker Swarm က Docker Engine တွေကို cluster အဖြစ်ပေါင်းပြီး **single virtual system** တစ်ခုလို manage လုပ်ပေးပါတယ်။

ဥပမာ—

```text
Server 1 + Server 2 + Server 3 = Docker Swarm Cluster
```

ဒီ cluster ထဲမှာ application တစ်ခုကို run လိုက်ရင် Docker Swarm က ဘယ် node ပေါ်မှာ container run မလဲဆိုတာကို automatically ဆုံးဖြတ်ပေးပါတယ်။

Docker Swarm က ကူညီပေးတာတွေ—

- Containers တွေကို scale လုပ်နိုင်တယ်
- Load balancing ပါပြီးသားဖြစ်တယ်
- Node/container fail ဖြစ်ရင် recover လုပ်ပေးနိုင်တယ်
- High availability ရနိုင်တယ်
- Docker commands တွေနဲ့ပဲ simple manage လုပ်နိုင်တယ်

---

# 2. Docker Swarm Architecture

Docker Swarm မှာ node အမျိုးအစား ၂ မျိုးရှိပါတယ်။

1. Manager Node
2. Worker Node

---

## 2.1 Manager Node

**Manager Node** က cluster ရဲ့ control center ပါ။

Manager node လုပ်ပေးတာတွေက—

- Swarm cluster ကို manage လုပ်တယ်
- Services တွေ schedule လုပ်တယ်
- Worker nodes တွေကို tasks assign လုပ်တယ်
- Desired state ကို maintain လုပ်တယ်
- Worker/container fail ဖြစ်ရင် task အသစ်ပြန်ဖန်တီးပေးတယ်
- Leader election နဲ့ high availability ထိန်းပေးတယ်

ဥပမာ service ကို replicas 3 ခု run ခိုင်းထားရင် manager က container 3 ခု အမြဲ run နေအောင် စောင့်ကြည့်ပေးပါတယ်။

---

## 2.2 Worker Node

**Worker Node** က actual containers တွေ run ပေးတဲ့ machine ပါ။

Worker node လုပ်ပေးတာတွေက—

- Manager က assign လုပ်တဲ့ task တွေကို run တယ်
- Containers/services တွေ run တယ်
- Task status ကို manager ဆီ report ပြန်ပို့တယ်

Architecture ပုံစံ—

```text
Manager Node
   ├── Worker Node 1 → Container run
   ├── Worker Node 2 → Container run
   └── Worker Node 3 → Container run
```

---

## 2.3 Raft Consensus

Docker Swarm manager nodes တွေက **Raft consensus** ကိုသုံးပြီး cluster state ကို consistent ဖြစ်အောင် ထိန်းပါတယ်။

အဓိကတာဝန်တွေက—

- Leader manager ရွေးချယ်ခြင်း
- Cluster state consistency ထိန်းခြင်း
- High availability ထောက်ပံ့ခြင်း

Production မှာ manager node တစ်လုံးထက်ပိုထားရင် failover ပိုကောင်းပါတယ်။

---

## 2.4 Overlay Network

Docker Swarm မှာ **overlay network** ကိုသုံးပြီး node မတူတဲ့ containers တွေ communicate လုပ်နိုင်ပါတယ်။

```text
Container on Worker Node 1  ↔  Container on Worker Node 2
```

ဒီ communication က Swarm cluster အတွင်းမှာ secure ဖြစ်အောင် design လုပ်ထားပါတယ်။

---

# 3. Docker Swarm ဘာကြောင့်သုံးလဲ?

## 3.1 Easy Scaling

Application replicas ကို လွယ်လွယ်ကူကူ တိုး/လျှော့နိုင်ပါတယ်။

```bash
docker service scale my-web=5
```

ဒီ command က `my-web` service ကို replicas 5 ခုအဖြစ် scale လုပ်ပေးပါတယ်။

---

## 3.2 Built-in Load Balancing

Docker Swarm မှာ load balancing ပါပြီးသားပါ။

```text
User Request
     ↓
Swarm Load Balancer
     ↓
Container 1 / Container 2 / Container 3
```

Service ကို replicas 3 ခုနဲ့ run ထားရင် user request တွေကို running containers တွေဆီ automatically distribute လုပ်ပေးပါတယ်။

---

## 3.3 High Availability

Node တစ်ခု fail ဖြစ်သွားရင် Swarm က container/task ကို တခြား available node ပေါ်မှာ ပြန် run ပေးနိုင်ပါတယ်။

```text
Worker Node 1 fail
↓
Swarm detects failure
↓
Task ကို Worker Node 2 ပေါ်မှာ ပြန် run
```

---

## 3.4 Self-healing

Docker Swarm က desired state ကို အမြဲ maintain လုပ်ပါတယ်။

ဥပမာ desired replicas = 3 ဖြစ်ပြီး container တစ်ခု crash ဖြစ်သွားရင်—

```text
Current replicas = 2
Desired replicas = 3
Swarm creates 1 new container
```

ဒီလို self-healing လုပ်ပေးတာကြောင့် service reliability ပိုကောင်းပါတယ်။

---

## 3.5 Simple Management

Docker Swarm က Kubernetes ထက် setup လုပ်ရတာ ပိုလွယ်ပါတယ်။

```bash
docker swarm init
docker service create
docker service ls
docker service scale
docker service update
```

---

# 4. Docker Swarm Today

Docker Swarm က powerful ဖြစ်ပေမယ့် သူ့ scope နဲ့ limitations ရှိပါတယ်။

Docker Swarm က simple, lightweight, easy to learn ဖြစ်ပါတယ်။  
ဒါပေမယ့် Kubernetes လောက် large-scale production ecosystem မကြီးပါဘူး။

---

## 4.1 Docker Swarm သင့်တော်တဲ့နေရာ

Docker Swarm သုံးဖို့သင့်တော်တဲ့ use cases တွေက—

- Small to medium applications
- Internal tools
- Simple microservices
- On-premise servers
- Edge environments
- Docker already သုံးနေတဲ့ team
- Learning container orchestration
- Kubernetes လောက် complexity မလိုတဲ့ project

---

## 4.2 Docker Swarm မသင့်တော်နိုင်တဲ့နေရာ

အောက်ပါအခြေအနေတွေမှာ Docker Swarm ထက် Kubernetes က ပိုသင့်တော်နိုင်ပါတယ်။

- Very large-scale production systems
- Complex cloud-native architecture
- Multi-cloud / hybrid cloud complex setup
- Advanced networking လိုတဲ့ project
- Service mesh/operator ecosystem လိုတဲ့ project
- Enterprise-level Kubernetes integrations လိုတဲ့ project

---

# 5. Initialize Swarm Cluster

Docker Swarm cluster တစ်ခုစဖို့ manager node ပေါ်မှာ `docker swarm init` run လုပ်ရပါတယ်။

```bash
docker swarm init --advertise-addr 192.168.1.10
```

`--advertise-addr` ဆိုတာ worker nodes တွေက manager node ကိုဆက်သွယ်ဖို့သုံးမယ့် IP address ပါ။

---

## 5.1 Swarm Init လုပ်တဲ့အခါ ဘာဖြစ်လဲ?

`docker swarm init` run လုပ်လိုက်ရင်—

- Current node က manager node ဖြစ်သွားတယ်
- Unique cluster ID တစ်ခု generate ဖြစ်တယ်
- Worker nodes join လုပ်ဖို့ token ထွက်လာတယ်
- Manager node က workers တွေကို accept လုပ်ဖို့ ready ဖြစ်သွားတယ်

Example output—

```text
Swarm initialized: current node is now a manager.

To add a worker to this swarm, run the following command:

docker swarm join --token SWMTKN-1-xxxxxxxxxxxxxxxx 192.168.1.10:2377
```

---

# 6. Join Worker Nodes

Swarm cluster initialize လုပ်ပြီးရင် worker nodes တွေကို cluster ထဲ join လုပ်နိုင်ပါတယ်။

Worker node တစ်ခုချင်းစီမှာ အောက်ပါ command run ပါ။

```bash
docker swarm join --token SWMTKN-1-xxxxxxxxxxxxxxxx 192.168.1.10:2377
```

---

## 6.1 Worker Join လုပ်တဲ့အခါ ဘာဖြစ်လဲ?

Worker node join လုပ်တဲ့အခါ—

- Worker က manager ကို authenticate လုပ်တယ်
- Worker node က cluster ထဲပေါ်လာတယ်
- Manager က ဒီ worker ပေါ်မှာ task schedule လုပ်နိုင်သွားတယ်
- Worker က containers run ပြီး status report ပြန်ပို့တယ်

---

## 6.2 Join Token ကို Manager မှာ ပြန်ကြည့်ခြင်း

Worker join token မေ့သွားရင် manager node ပေါ်မှာ run ပါ။

```bash
docker swarm join-token worker
```

Join command format—

```bash
docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```

---

# 7. Service ဆိုတာဘာလဲ?

Docker Swarm မှာ app ကို container တစ်ခုချင်းစီအနေနဲ့ run တာထက် **service** အနေနဲ့ run ပါတယ်။

Service ဆိုတာ app ကို ဘယ်လို run မလဲဆိုတာ define လုပ်ထားတဲ့ object ပါ။

Service ထဲမှာ သတ်မှတ်နိုင်တာတွေက—

- Image name
- Replicas count
- Published ports
- Networks
- Environment variables
- Update policy
- Restart policy

---

## 7.1 Service နဲ့ Container ကွာခြားချက်

| Topic | Container | Service |
|---|---|---|
| Scope | Single container | Desired state for multiple tasks |
| Scaling | Manual | Built-in replicas |
| Self-healing | မပါ | ပါ |
| Load balancing | Manual setup လိုနိုင် | Built-in |
| Swarm support | Basic | Main object |

---

# 8. Deploy Your First Service

ဥပမာ nginx service တစ်ခု deploy လုပ်မယ်။

```bash
docker service create \
  --name my-web \
  --replicas 3 \
  --publish 8080:80 \
  nginx:alpine
```

Command ရှင်းပြချက်—

```text
--name my-web       = service name
--replicas 3        = container/task 3 ခု run
--publish 8080:80   = host port 8080 ကို container port 80 နဲ့ map
nginx:alpine        = အသုံးပြုမယ့် image
```

---

## 8.1 Service Create လုပ်ပြီးရင် ဘာဖြစ်လဲ?

Swarm က—

1. Service ကို create လုပ်တယ်
2. Replicas 3 ခု run ဖို့ desired state မှတ်ထားတယ်
3. Scheduler က available nodes တွေရှာတယ်
4. Worker nodes တွေပေါ်မှာ tasks တွေ place လုပ်တယ်
5. Service ကို running state ဖြစ်အောင် maintain လုပ်တယ်

---

# 9. Routing Mesh

Docker Swarm မှာ service port ကို publish လုပ်ထားရင် cluster ထဲက node ဘယ် node ကို request ပို့ပို့ service ကို access လုပ်လို့ရပါတယ်။

ဥပမာ service ကို `8080:80` နဲ့ publish ထားတယ်ဆိုရင်—

```text
http://manager-ip:8080  → works
http://worker1-ip:8080  → works
http://worker2-ip:8080  → works
```

Container က worker1 ပေါ်မှာပဲ run နေရင်တောင် worker2 IP ကနေဝင်လို့ရနိုင်ပါတယ်။  
ဒါကို **Routing Mesh** လို့ခေါ်ပါတယ်။

---

# 10. Built-in Load Balancing and Service Discovery

Docker Swarm မှာ service discovery နဲ့ load balancing ပါပြီးသားဖြစ်ပါတယ်။

Service name နဲ့ container တွေ communicate လုပ်နိုင်ပါတယ်။

```text
backend service → database service
```

Service name ကို DNS name လိုသုံးနိုင်ပါတယ်။

---

# 11. Scale Service

Service replicas ကို တိုးချင်ရင်—

```bash
docker service scale my-web=5
```

Replicas ကို လျှော့ချင်ရင်—

```bash
docker service scale my-web=2
```

Swarm က available nodes တွေပေါ်မှာ tasks တွေကို automatically distribute လုပ်ပေးပါတယ်။

---

# 12. Rolling Update

Docker Swarm မှာ service image ကို update လုပ်တဲ့အခါ containers အကုန်လုံးကို တစ်ခါတည်းမပြောင်းဘဲ တဖြည်းဖြည်း update လုပ်နိုင်ပါတယ်။

```bash
docker service update \
  --image nginx:1.25-alpine \
  --update-parallelism 1 \
  --update-delay 10s \
  my-web
```

Command ရှင်းပြချက်—

```text
--image nginx:1.25-alpine  = image version အသစ်
--update-parallelism 1     = တစ်ကြိမ်မှာ task 1 ခုပဲ update
--update-delay 10s         = update တစ်ခုနဲ့တစ်ခုကြား 10 seconds စောင့်
my-web                     = update လုပ်မယ့် service
```

Rolling update flow—

1. Old task တစ်ခုကို stop လုပ်တယ်
2. New task တစ်ခုကို start လုပ်တယ်
3. Health check pass ဖြစ်မဖြစ်စစ်တယ်
4. Old task ကို remove လုပ်တယ်
5. နောက် task ကို ဆက် update လုပ်တယ်
6. Tasks အကုန် update ပြီးတဲ့အထိ repeat လုပ်တယ်

---

# 13. Manage Replicas in Compose Using Swarm

Docker Compose file ကို Swarm မှာသုံးချင်ရင် `deploy.replicas` ကိုသုံးပါတယ်။

Example `docker-compose.yml`:

```yaml
version: "3.8"

services:
  backend:
    image: myapp/backend:latest
    ports:
      - "5001:5000"
    environment:
      APP_NAME: ${APP_NAME}
      DB_HOST: ${MYSQL_HOST}
      DB_PORT: ${MYSQL_PORT}
      DB_NAME: ${MYSQL_DATABASE}
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
    networks:
      - app-net
    deploy:
      replicas: 3
      restart_policy:
        condition: any

  mysql:
    image: mysql:8.0
    ports:
      - "${MYSQL_PORT}:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app-net
    deploy:
      replicas: 1

  redis:
    image: redis:7-alpine
    ports:
      - "${REDIS_PORT}:6379"
    networks:
      - app-net
    deploy:
      replicas: 1

networks:
  app-net:
    driver: overlay

volumes:
  mysql_data:
```

---

## 13.1 Stack Commands

Compose file ကို Swarm stack အနေနဲ့ deploy လုပ်ရန်—

```bash
docker stack deploy -c docker-compose.yml myapp
```

Stack services ကြည့်ရန်—

```bash
docker stack services myapp
```

Backend service ကို scale လုပ်ရန်—

```bash
docker service scale myapp_backend=5
```

Backend service ကို 2 replicas ပြန်လျှော့ရန်—

```bash
docker service scale myapp_backend=2
```

Service inspect ကြည့်ရန်—

```bash
docker service inspect myapp_backend
```

Stack remove လုပ်ရန်—

```bash
docker stack rm myapp
```

---

# 14. Best Practices for Replicas

Docker Swarm မှာ replicas သုံးတဲ့အခါ—

- Stateless services တွေအတွက် replicas တိုးသုံးပါ
- Stateful services တွေဖြစ်တဲ့ MySQL/Redis ကို replicas 1 ထားတာပိုသင့်တော်ပါတယ်
- Zero-downtime update အတွက် `update_config` သုံးပါ
- Swarm က desired replica count ကို အမြဲ maintain လုပ်ပေးပါတယ်
- Advanced placement လိုရင် placement constraints သုံးနိုင်ပါတယ်

---

# 15. Docker Swarm vs Kubernetes

| Topic | Docker Swarm | Kubernetes |
|---|---|---|
| Setup | လွယ် | ပိုခက် |
| Learning curve | နည်း | မြင့် |
| Docker integration | Built-in | Separate ecosystem |
| Large-scale production | Limited | အရမ်းကောင်း |
| Ecosystem | သေး | အကြီးဆုံး |
| Networking | Simple | Advanced |
| Best for | Small/medium apps | Enterprise/cloud-native apps |

---

# 16. Important Commands Summary

## Swarm initialize

```bash
docker swarm init --advertise-addr <MANAGER-IP>
```

## Worker join

```bash
docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```

## Worker join token ကြည့်ရန်

```bash
docker swarm join-token worker
```

## Nodes list ကြည့်ရန်

```bash
docker node ls
```

## Service create

```bash
docker service create --name my-web --replicas 3 --publish 8080:80 nginx:alpine
```

## Services list ကြည့်ရန်

```bash
docker service ls
```

## Service tasks ကြည့်ရန်

```bash
docker service ps my-web
```

## Service scale

```bash
docker service scale my-web=5
```

## Service update

```bash
docker service update --image nginx:1.25-alpine my-web
```

## Service remove

```bash
docker service rm my-web
```

## Stack deploy

```bash
docker stack deploy -c docker-compose.yml myapp
```

## Stack services ကြည့်ရန်

```bash
docker stack services myapp
```

## Stack remove

```bash
docker stack rm myapp
```

---

# 17. Final Summary

Docker Swarm ဆိုတာ Docker hosts အများကြီးကို cluster အဖြစ်ပေါင်းပြီး containers တွေကို **services** အနေနဲ့ manage လုပ်ပေးတဲ့ Docker native orchestration tool ပါ။

အဓိက concept က—

```text
Manager node က cluster ကို control လုပ်တယ်
Worker nodes တွေက containers run တယ်
Service က desired state ကို define လုပ်တယ်
Replicas က app instances အရေအတွက်ကို သတ်မှတ်တယ်
Swarm က scaling, load balancing, self-healing, rolling update တွေကို handle လုပ်ပေးတယ်
```

လွယ်လွယ်မှတ်ရရင်—

**Docker Swarm = Docker containers တွေကို server အများကြီးပေါ်မှာ လွယ်လွယ်ကူကူ run, scale, update, recover လုပ်ပေးတဲ့ cluster management tool ပါ။**
