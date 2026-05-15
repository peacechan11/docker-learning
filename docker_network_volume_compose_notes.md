# Docker Network, Volume & Docker Compose Notes

> ဒီ note က Docker Network, Docker Volume, Docker Compose အကြောင်းကို beginner-friendly မြန်မာလို ရှင်းပြထားတာပါ။  
> Container တွေ အချင်းချင်း communicate လုပ်ဖို့ **network** လိုပြီး၊ data မပျက်အောင် သိမ်းဖို့ **volume** လိုပါတယ်။ Multiple containers ကို တစ်ခါတည်း run/manage လုပ်ချင်ရင် **Docker Compose** ကိုသုံးပါတယ်။

---

# Part 1 — Docker Network

## 1. Docker Network ဆိုတာဘာလဲ?

Docker Network ဆိုတာ container တွေကို အချင်းချင်း communicate လုပ်နိုင်အောင် ချိတ်ဆက်ပေးတဲ့ virtual network layer တစ်ခုပါ။

Container တစ်ခုနဲ့ တစ်ခု data ပို့ချင်ရင် network တစ်ခုထဲမှာရှိဖို့လိုပါတယ်။

```text
Container A  <---- Docker Network ---->  Container B
```

Docker network က—

- Container တွေကြား communication လုပ်ပေးတယ်။
- Container traffic ကို isolate လုပ်ပေးတယ်။
- Internal DNS ကိုသုံးပြီး container name နဲ့ချိတ်လို့ရစေတယ်။
- External network/internet ကို access လုပ်နိုင်စေတယ်။

---

## 2. Docker Network Types

| Network Type | Meaning | Use Case |
|---|---|---|
| `bridge` | Default network driver | Single host မှာ containers တွေချိတ်ဖို့ |
| `host` | Container က host network ကိုတိုက်ရိုက်သုံး | Performance လိုတဲ့ case |
| `none` | Network disabled | Network မလိုတဲ့ isolated container |
| `overlay` | Multiple Docker hosts ကြား network ချိတ် | Docker Swarm / multi-host |

---

## 3. Bridge Network

`bridge` network က Docker ရဲ့ default network ဖြစ်ပါတယ်။ Container တွေကို private internal network တစ်ခုထဲမှာထားပြီး အချင်းချင်း communicate လုပ်စေပါတယ်။

```text
Container A: 172.18.0.2
Container B: 172.18.0.3
Both are connected to docker0 bridge.
```

Bridge network မှာ—

- Container တစ်ခုချင်းစီမှာ private IP ရှိတယ်။
- Same bridge network ထဲက containers တွေ communicate လုပ်နိုင်တယ်။
- Host machine က `docker0` bridge မှတစ်ဆင့် container တွေနဲ့ communicate လုပ်နိုင်တယ်။

---

## 4. Host / None / Overlay Network

### Host Network

```bash
docker run --network host nginx
```

Container က host machine ရဲ့ network stack ကို share လုပ်ပါတယ်။ Network isolation မရှိတော့ပေမယ့် performance ကောင်းနိုင်ပါတယ်။

### None Network

```bash
docker run --network none ubuntu
```

Container ကို network မပေးပါဘူး။ Security isolation လိုတဲ့ task တွေအတွက်သုံးနိုင်ပါတယ်။

### Overlay Network

Multiple Docker hosts ကြား containers တွေ communicate လုပ်ဖို့သုံးပါတယ်။ Docker Swarm / multi-host setup တွေမှာအသုံးများပါတယ်။

---

## 5. Container Name နဲ့ Communicate လုပ်ခြင်း

Docker network ထဲမှာ container name ကို hostname လိုသုံးနိုင်ပါတယ်။

ဥပမာ `web` container က `db` container ကိုချိတ်မယ်ဆိုရင်—

```env
DB_HOST=db
```

ဒီနေရာမှာ `db` က database container name/service name ဖြစ်ပါတယ်။

**Important:**  
Container ထဲက `localhost` ဆိုတာ host machine ကိုမဆိုလိုပါဘူး။ Container ကိုယ်တိုင်ကိုပဲဆိုလိုပါတယ်။

မှားနိုင်တဲ့ example—

```env
DB_HOST=localhost
```

မှန်တဲ့ example—

```env
DB_HOST=db
```

---

## 6. Useful Docker Network Commands

Docker network list ကြည့်ရန်—

```bash
docker network ls
```

Bridge network အသေးစိတ်ကြည့်ရန်—

```bash
docker network inspect bridge
```

Custom network create လုပ်ရန်—

```bash
docker network create my-net
```

Container ကို custom network နဲ့ run လုပ်ရန်—

```bash
docker run -d --name web --network my-net nginx
```

Network remove လုပ်ရန်—

```bash
docker network rm my-net
```

Unused networks cleanup လုပ်ရန်—

```bash
docker network prune
```

---

## 7. Custom Bridge Network Example

Custom network တစ်ခု create လုပ်ပါ—

```bash
docker network create app-net
```

Nginx container run—

```bash
docker run -d --name web --network app-net nginx
```

Busybox container ကနေ `web` ကို ping လုပ်ကြည့်ပါ—

```bash
docker run --rm --network app-net busybox ping web
```

ဒီမှာ `web` ဆိုတာ container name ဖြစ်ပြီး Docker internal DNS က IP address ကို resolve လုပ်ပေးပါတယ်။

---

# Part 2 — Docker Volume

## 8. Docker Volume ဆိုတာဘာလဲ?

Docker Volume ဆိုတာ container data ကို host machine ပေါ်မှာ persist လုပ်ဖို့ Docker က manage လုပ်ပေးတဲ့ storage mechanism ပါ။

Container ကို remove လုပ်လိုက်ရင် container ထဲက data ပျက်နိုင်ပါတယ်။ ဒါပေမယ့် volume သုံးထားရင် data က volume ထဲမှာကျန်နေပါတယ်။

Volume ကို database data, uploads, logs, app data တွေသိမ်းဖို့သုံးပါတယ်။

---

## 9. Volume ဘာလို့လိုလဲ?

Container တွေက temporary ဖြစ်ပါတယ်။

```text
Container removed  ->  container filesystem data can be lost
Volume used        ->  data survives container restart/removal
```

ဥပမာ MySQL container မှာ database data ကို `/var/lib/mysql` ထဲသိမ်းပါတယ်။  
ဒီ folder ကို volume နဲ့ချိတ်ထားရင် MySQL container ဖျက်ပြီးပြန် run လုပ်လည်း data မပျက်ပါဘူး။

---

## 10. Named Volume Example

Volume create လုပ်ရန်—

```bash
docker volume create mysql_data
```

MySQL container run—

```bash
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=compose_demo \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=apppassword \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0
```

ဒီ command မှာ—

```text
mysql_data:/var/lib/mysql
```

| Part | Meaning |
|---|---|
| `mysql_data` | Host side Docker named volume |
| `/var/lib/mysql` | Container ထဲက MySQL data directory |

---

## 11. Docker Volume Commands

Existing volumes ကြည့်ရန်—

```bash
docker volume ls
```

Volume အသေးစိတ်ကြည့်ရန်—

```bash
docker volume inspect mysql_data
```

Sample output—

```json
{
  "Name": "mysql_data",
  "Driver": "local",
  "Mountpoint": "/var/lib/docker/volumes/mysql_data/_data"
}
```

Volume remove လုပ်ရန်—

```bash
docker volume rm mysql_data
```

Unused volumes cleanup လုပ်ရန်—

```bash
docker volume prune
```

**Warning:**  
Volume ထဲမှာ database data, upload files, important logs တွေရှိနိုင်လို့ remove/prune မလုပ်ခင် သေချာစစ်ပါ။

---

## 12. Bind Mount vs Named Volume

| Type | Example | Meaning |
|---|---|---|
| Named Volume | `mysql_data:/var/lib/mysql` | Docker က manage လုပ်တဲ့ volume |
| Bind Mount | `/Users/aries/mysql_data:/var/lib/mysql` | Host path ကိုတိုက်ရိုက် mount လုပ်ခြင်း |

Named volume example—

```bash
-v mysql_data:/var/lib/mysql
```

Bind mount example—

```bash
-v /Users/aries/mysql_data:/var/lib/mysql
```

### Named Volume ကို ဘယ်အချိန်သုံးမလဲ?

- Database data persist လုပ်ချင်တဲ့အခါ
- Docker က storage location ကို manage လုပ်စေချင်တဲ့အခါ
- Production-like setup

### Bind Mount ကို ဘယ်အချိန်သုံးမလဲ?

- Local development မှာ source code ကို container ထဲ mount လုပ်ချင်တဲ့အခါ
- Host machine ပေါ်က exact folder ကိုသုံးချင်တဲ့အခါ
- File changes ကို container ထဲမှာချက်ချင်းမြင်ချင်တဲ့အခါ

---

# Part 3 — Docker Compose

## 13. Docker Compose ဆိုတာဘာလဲ?

Docker Compose ဆိုတာ multiple containers ကို `docker-compose.yml` သို့မဟုတ် `compose.yml` file တစ်ခုနဲ့ define လုပ်ပြီး တစ်ခါတည်း run/manage လုပ်နိုင်တဲ့ tool ပါ။

ဥပမာ real application တစ်ခုမှာ—

```text
Frontend container
Backend API container
Database container
Redis container
```

ဒီ containers တွေကို `docker run` command အများကြီးနဲ့ run မလုပ်တော့ဘဲ Compose file ထဲမှာရေးပြီး—

```bash
docker compose up
```

တစ်ကြောင်းနဲ့ run လုပ်နိုင်ပါတယ်။

---

## 14. Docker Compose ဘာလို့သုံးသင့်လဲ?

Docker Compose က—

- Multiple containers ကို တစ်နေရာတည်းမှာ manage လုပ်နိုင်တယ်။
- Network ကို auto create လုပ်ပေးတယ်။
- Service name နဲ့ container အချင်းချင်း communicate လုပ်နိုင်တယ်။
- Volumes, ports, environment variables တွေကို YAML file ထဲမှာရှင်းရှင်းလင်းလင်းရေးနိုင်တယ်။
- Development environment setup လုပ်ရလွယ်တယ်။

---

## 15. Basic Docker Compose Structure

`docker-compose.yml` example—

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
```

Run လုပ်ရန်—

```bash
docker compose up -d
```

Stop/remove လုပ်ရန်—

```bash
docker compose down
```

---

## 16. Docker Compose with MySQL Volume Example

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: compose-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: compose_demo
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppassword
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

Run—

```bash
docker compose up -d
```

Check containers—

```bash
docker compose ps
```

View logs—

```bash
docker compose logs mysql
```

Stop/remove containers—

```bash
docker compose down
```

Stop/remove containers + volumes—

```bash
docker compose down -v
```

**Warning:**  
`docker compose down -v` က volumes တွေပါဖျက်နိုင်တာကြောင့် database data ပျက်နိုင်ပါတယ်။

---

## 17. Docker Compose Network

Docker Compose က default network တစ်ခုကို automatically create လုပ်ပေးပါတယ်။

```yaml
services:
  backend:
    build: .
    ports:
      - "8080:8080"

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
```

ဒီ setup မှာ `backend` container က database ကို `localhost` နဲ့မခေါ်ပါဘူး။  
Service name ဖြစ်တဲ့ `db` နဲ့ခေါ်ရပါတယ်။

```env
DB_HOST=db
DB_PORT=3306
DB_USER=root
DB_PASSWORD=rootpassword
```

---

## 18. Backend + MySQL Docker Compose Example

```yaml
services:
  backend:
    build: .
    container_name: backend-api
    ports:
      - "8080:8080"
    environment:
      DB_HOST: mysql
      DB_PORT: 3306
      DB_NAME: app_db
      DB_USER: appuser
      DB_PASSWORD: apppassword
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    container_name: mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: app_db
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppassword
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

ဒီမှာ backend က MySQL ကို ဒီလိုချိတ်ရပါမယ်—

```env
DB_HOST=mysql
```

`mysql` ဆိုတာ Compose service name ဖြစ်ပါတယ်။

---

## 19. Docker Compose Commands

| Purpose | Command |
|---|---|
| Start services | `docker compose up` |
| Start in background | `docker compose up -d` |
| Stop/remove services | `docker compose down` |
| Stop/remove services + volumes | `docker compose down -v` |
| View running services | `docker compose ps` |
| View logs | `docker compose logs` |
| Follow logs | `docker compose logs -f` |
| View one service log | `docker compose logs mysql` |
| Restart services | `docker compose restart` |
| Build images | `docker compose build` |
| Rebuild and start | `docker compose up --build -d` |
| Execute command in service | `docker compose exec backend sh` |
| Stop services only | `docker compose stop` |
| Start stopped services | `docker compose start` |

---

## 20. Docker Compose vs Docker Run

| Docker Run | Docker Compose |
|---|---|
| Single container run လုပ်ဖို့ပိုသင့် | Multiple containers manage ဖို့ပိုသင့် |
| Command ရှည်နိုင် | YAML file နဲ့သန့်ရှင်း |
| Network/volume ကို manual setup လုပ်ရနိုင် | Compose က auto manage လုပ်ပေး |
| Small test အတွက်ကောင်း | Real app development အတွက်ကောင်း |

---

## 21. Practical Flow

```text
1. Dockerfile ရေး
2. docker-compose.yml ရေး
3. services, ports, env, volumes သတ်မှတ်
4. docker compose up -d
5. docker compose ps နဲ့စစ်
6. docker compose logs နဲ့ debug
7. docker compose down နဲ့ stop/remove
```

---

# Quick Summary

## Docker Network

- Containers တွေ communicate လုပ်ဖို့သုံးတယ်။
- Default network က `bridge` ဖြစ်တယ်။
- Same network ထဲမှာ container name/service name နဲ့ခေါ်လို့ရတယ်။
- Container ထဲက `localhost` က container ကိုယ်တိုင်ကိုဆိုလိုတယ်။

## Docker Volume

- Container data persist လုပ်ဖို့သုံးတယ်။
- Database data, uploads, logs တွေအတွက်အရေးကြီးတယ်။
- Named volume ကို Docker က manage လုပ်ပေးတယ်။
- Volume prune/remove လုပ်ရင် data ပျက်နိုင်လို့ သတိထားရမယ်။

## Docker Compose

- Multiple containers ကို YAML file တစ်ခုနဲ့ manage လုပ်တာပါ။
- Network, volume, env, ports တွေကို တစ်နေရာတည်းမှာရေးနိုင်တယ်။
- Service name နဲ့ containers တွေ communicate လုပ်နိုင်တယ်။
- Development environment တစ်ခုလုံးကို `docker compose up -d` တစ်ကြောင်းနဲ့ run နိုင်တယ်။

---

## Final Key Takeaway

Docker Network က containers တွေကိုချိတ်ဆက်ပေးပါတယ်။  
Docker Volume က container data ကိုမပျက်အောင်သိမ်းပေးပါတယ်။  
Docker Compose က multiple containers, networks, volumes, environment variables တွေကို file တစ်ခုထဲမှာရေးပြီး တစ်ခါတည်း run/manage လုပ်နိုင်စေပါတယ်။
