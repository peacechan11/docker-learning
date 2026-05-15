# Docker Advanced Commands Notes

ဒီ note က Docker ကို basic run/manage လုပ်နိုင်ပြီးနောက်ပိုင်း သိထားသင့်တဲ့ **logs, inspect, stats, file operations, Docker Hub, image tagging/pushing, cleanup** commands တွေကို မြန်မာလို အတိုချုံးမှတ်ထားတာပါ။

---

## 1. Docker Advanced Logs

Docker container ထဲက application output / error / runtime log တွေကို ကြည့်ဖို့ `docker logs` ကိုသုံးပါတယ်။

### 1.1 View Logs

Container တစ်ခုရဲ့ logs ကြည့်ရန်—

```bash
$ docker logs container_name
```

ဥပမာ—

```bash
$ docker logs web-server
/docker-entrypoint.sh: Configuration complete; ready for start up
```

### 1.2 Follow Logs `-f`

Logs ကို real-time stream အနေနဲ့ကြည့်ရန်—

```bash
$ docker logs -f container_name
```

App run နေတုန်း live log ကြည့်ချင်ရင် သုံးပါတယ်။

### 1.3 Tail Logs

နောက်ဆုံး N lines ပဲကြည့်ချင်ရင်—

```bash
$ docker logs --tail 100 container_name
```

နောက်ဆုံး 100 lines ကိုပဲပြပေးပါတယ်။

### 1.4 Filter by Time

သတ်မှတ်ထားတဲ့အချိန်နောက်ပိုင်း logs တွေပဲကြည့်ချင်ရင်—

```bash
$ docker logs --since 2024-05-01T10:00:00 container_name
```

### 1.5 Combine Options

နောက်ဆုံး 100 lines ကိုပြပြီး real-time follow လုပ်ချင်ရင်—

```bash
$ docker logs -f --tail 100 container_name
```

**မှတ်ရန်**

- `-f` = logs ကို real-time ကြည့်ရန်
- `--tail` = ပြချင်တဲ့ line အရေအတွက် limit လုပ်ရန်
- Logs တွေက default အနေနဲ့ container runtime ထဲမှာပဲရှိတာကြောင့် persistent မဖြစ်နိုင်ပါ

---

## 2. Docker Inspect

`docker inspect` က container/image/network/volume တစ်ခုရဲ့ detailed information ကို **JSON format** နဲ့ပြပေးပါတယ်။

### 2.1 Inspect Container

```bash
$ docker inspect container_name_or_id
```

ဥပမာ—

```bash
$ docker inspect my-nginx
[
  {
    "Id": "f6a1c2d3e4b5...",
    "Name": "/my-nginx",
    "Image": "nginx:latest",
    "State": {
      "Status": "running",
      "Running": true
    },
    "Config": {
      "Hostname": "f6a1c2d3e4b5",
      "Env": [
        "NGINX_VERSION=1.25.3"
      ]
    }
  }
]
```

`docker inspect` မှာ သိနိုင်တာတွေ—

| Information | Meaning |
|---|---|
| Container ID & Name | Container identity |
| Image information | ဘယ် image ကနေ run ထားလဲ |
| State & Status | running / exited စတဲ့ state |
| Configuration | command, env, hostname စတာတွေ |
| Network Settings | IP address, port mapping |
| Mounted Volumes | volume mount information |
| Environment Variables | container env values |

### 2.2 Inspect Specific Field

JSON ထဲက field တစ်ခုတည်းကိုပဲထုတ်ချင်ရင် `--format` သုံးနိုင်ပါတယ်။

```bash
$ docker inspect -f '{{.State.Status}}' my-nginx
running
```

**မှတ်ရန်**  
Full JSON မလိုဘဲ status, IP, env, mount path စတာတစ်ခုချင်းလိုချင်ရင် `--format` က အသုံးဝင်ပါတယ်။

---

## 3. Monitor Performance

Running containers တွေရဲ့ CPU, Memory, Network I/O, Block I/O usage ကို real-time ကြည့်ဖို့ `docker stats` ကိုသုံးပါတယ်။

```bash
$ docker stats
CONTAINER ID   NAME          CPU %   MEM USAGE / LIMIT   MEM %   NET I/O        BLOCK I/O       PIDS
a1b2c3d4e5f6   web-app       2.45%   45.6MiB / 512MiB    8.91%   1.2kB / 3.4kB   2.1MB / 1.2MB   12
d4e5f6g7h8i9   api-service   1.12%   32.1MiB / 512MiB    6.27%   2.3kB / 1.1kB   1.3MB / 980kB   9
```

ကြည့်နိုင်တဲ့ metrics တွေ—

| Metric | Meaning |
|---|---|
| CPU % | Container CPU usage |
| MEM USAGE / LIMIT | Memory usage and limit |
| MEM % | Memory percentage |
| NET I/O | Network input/output |
| BLOCK I/O | Disk read/write usage |
| PIDS | Process count |

Stats stream ကိုရပ်ချင်ရင်—

```bash
Ctrl + C
```

**အသုံးများတဲ့အချိန်**

- Container resource usage စစ်ချင်တဲ့အခါ
- App slow ဖြစ်နေလားကြည့်ချင်တဲ့အခါ
- Memory leak / CPU spike ရှိလား monitor လုပ်ချင်တဲ့အခါ

---

## 4. Manage Files in Containers

Container နဲ့ local machine ကြား file copy လုပ်တာ၊ container ထဲက file changes စစ်တာ၊ changes တွေကို image အသစ်အဖြစ် save လုပ်တာတွေမှာ ဒီ commands တွေကိုသုံးပါတယ်။

### 4.1 `docker cp`

Local machine က file/folder ကို container ထဲ copy လုပ်ရန်—

```bash
$ docker cp ./app.py mycontainer:/app/app.py
Successfully copied 2.05kB to mycontainer:/app/app.py
```

Container ထဲက file ကို local machine ပြန် copy လုပ်ရန်—

```bash
$ docker cp mycontainer:/app/app.py ./app.py
```

### 4.2 `docker exec`

Running container ထဲမှာ command run လုပ်ရန်—

```bash
$ docker exec -it mycontainer bash
root@mycontainer:/app# ls -l
total 16
-rw-r--r--  1 root root 2050 May 20 10:30 app.py
drwxr-xr-x  2 root root 4096 May 20 10:30 logs
```

`bash` မရှိရင်—

```bash
$ docker exec -it mycontainer sh
```

### 4.3 `docker diff`

Container ထဲမှာ file system changes ဖြစ်ထားတာတွေကိုစစ်ရန်—

```bash
$ docker diff mycontainer
A /app/new_file.txt
C /app/app.py
D /app/old_file.txt
```

| Symbol | Meaning |
|---|---|
| A | Added file |
| C | Changed/modified file |
| D | Deleted file |

### 4.4 `docker commit`

Container ထဲမှာပြင်ထားတဲ့ changes တွေကို image အသစ်အဖြစ် save လုပ်ရန်—

```bash
$ docker commit mycontainer myapp:v1.0
sha256:5f3b2c1d7e8a9b0c6d4f...
```

**မှတ်ရန်**  
`docker commit` က learning/debug အတွက်အသုံးဝင်ပေမယ့် real project မှာ reproducible ဖြစ်အောင် Dockerfile နဲ့ build လုပ်တာပိုကောင်းပါတယ်။

---

## 5. Work with Docker Hub

Docker Hub က Docker images တွေကို upload/download/search လုပ်နိုင်တဲ့ public registry ဖြစ်ပါတယ်။

### 5.1 Docker Hub Login

```bash
$ docker login
Username: your_dockerhub_username
Password: ********
Login Succeeded
```

Docker Hub ကို push လုပ်မယ်ဆို login လိုပါတယ်။

### 5.2 Search Image

Docker Hub မှာ image ရှာရန်—

```bash
$ docker search nginx
NAME                         DESCRIPTION                    STARS    OFFICIAL   AUTOMATED
nginx                        Official build of Nginx.       19402    [OK]
nginxinc/nginx-unprivileged   Unprivileged NGINX Dockerfiles 210                 [OK]
bitnami/nginx                Bitnami nginx Docker Image     158                 [OK]
```

`OFFICIAL [OK]` ဖြစ်တဲ့ image တွေက official image ဖြစ်လို့ beginner အတွက်ပိုယုံကြည်ရပါတယ်။

### 5.3 Pull Image

Image download လုပ်ရန်—

```bash
$ docker pull nginx:latest
latest: Pulling from library/nginx
7c4f8a8c3f8a: Pull complete
Digest: sha256:7b4d...e2c7
Status: Downloaded newer image for nginx:latest
```

### 5.4 Logout

```bash
$ docker logout
Removing login credentials for https://index.docker.io/v1/
Logout Succeeded
```

---

## 6. Image Tagging & Pushing

Image ကို Docker Hub repository ထဲ push မလုပ်ခင် tag ပေးရပါတယ်။

Tag format က—

```text
username/repository:tag
```

ဥပမာ—

```text
yourdockerhubuser/nginx:my-v1
```

| Part | Meaning |
|---|---|
| username | Docker Hub username |
| repository | Repository name |
| tag | Version/name label |

### 6.1 Pull Image

```bash
$ docker pull nginx:latest
```

### 6.2 Tag Image

```bash
$ docker tag nginx:latest yourdockerhubuser/nginx:my-v1
```

Tag ဆိုတာ image ကို nickname/version label ပေးတာပါ။ Same image ကို different version tags အများကြီးပေးနိုင်ပါတယ်။

ဥပမာ—

```bash
$ docker tag nginx:latest yourdockerhubuser/nginx:v1.0
$ docker tag nginx:latest yourdockerhubuser/nginx:v2.0
```

### 6.3 Push Image

```bash
$ docker push yourdockerhubuser/nginx:my-v1
```

Push ပြီးသွားရင် Docker Hub repository မှာ image ကိုတွေ့နိုင်ပါတယ်။

**Image workflow**

```text
Pull image → Tag image → Push image
```

Real project workflow ကတော့—

```text
Build own image → Tag image → Push image
```

**မှတ်ရန်**

- Meaningful tag တွေသုံးပါ — `v1`, `v2.0.1`, `latest`
- Push မလုပ်ခင် tag format မှန်ရပါမယ်
- Docker Hub username မှန်ရပါမယ်

---

## 7. Advanced Cleanup

Docker environment ကို clean ဖြစ်အောင် unused images, stopped containers, unused volumes, build cache တွေကိုရှင်းပေးနိုင်ပါတယ်။

### 7.1 Remove Images

Specific image remove လုပ်ရန်—

```bash
$ docker rmi myapp:1.0
```

Dangling images remove လုပ်ရန်—

```bash
$ docker image prune -f
```

**သတိထားရန်**  
Removed images တွေကို recover လုပ်လို့မရပါ။ လိုအပ်သေးလားအရင်စစ်ပါ။

### 7.2 Remove Containers

Specific container remove လုပ်ရန်—

```bash
$ docker rm container_id
```

Stopped containers အားလုံး remove လုပ်ရန်—

```bash
$ docker container prune -f
```

`docker container prune` က stopped containers တွေကိုပဲ remove လုပ်ပါတယ်။ Running containers တွေကိုမဖျက်ပါဘူး။

### 7.3 Remove Volumes

Specific volume remove လုပ်ရန်—

```bash
$ docker volume rm volume_name
```

Unused volumes remove လုပ်ရန်—

```bash
$ docker volume prune -f
```

**သတိထားရန်**  
Volumes ထဲမှာ database data / app data တွေရှိနိုင်ပါတယ်။ Remove မလုပ်ခင် သေချာစစ်ပါ။

### 7.4 Clean Everything

Unused images, containers, networks, build cache, volumes တွေပါ cleanup လုပ်ရန်—

```bash
$ docker system prune -a -f --volumes
```

ဒီ command က unused data အများကြီးဖျက်နိုင်လို့ သတိထားသုံးပါ။

**Warning**  
`--volumes` ပါရင် unused volumes တွေပါဖျက်နိုင်ပါတယ်။ Database volume တွေရှိရင် data ပျက်နိုင်ပါတယ်။

---

## Quick Command Cheat Sheet

| Purpose | Command |
|---|---|
| View logs | `docker logs container_name` |
| Follow logs | `docker logs -f container_name` |
| Last 100 lines | `docker logs --tail 100 container_name` |
| Logs since time | `docker logs --since 2024-05-01T10:00:00 container_name` |
| Inspect container | `docker inspect container_name_or_id` |
| Inspect one field | `docker inspect -f '{{.State.Status}}' container_name` |
| Monitor performance | `docker stats` |
| Copy local to container | `docker cp ./app.py container:/app/app.py` |
| Copy container to local | `docker cp container:/app/app.py ./app.py` |
| Exec into container | `docker exec -it container_name bash` |
| View file changes | `docker diff container_name` |
| Commit container to image | `docker commit container_name image_name:tag` |
| Docker Hub login | `docker login` |
| Search image | `docker search nginx` |
| Pull image | `docker pull nginx:latest` |
| Tag image | `docker tag nginx:latest username/nginx:my-v1` |
| Push image | `docker push username/nginx:my-v1` |
| Logout | `docker logout` |
| Remove image | `docker rmi image_name:tag` |
| Remove stopped containers | `docker container prune -f` |
| Remove unused volumes | `docker volume prune -f` |
| Clean unused data | `docker system prune -a -f --volumes` |

---

## Final Key Takeaway

Docker advanced commands တွေမှာ အဓိကသိထားရမှာက—

```text
Logs → Inspect → Stats → File Operations → Docker Hub → Tag & Push → Cleanup
```

`docker logs` နဲ့ error/debug ကြည့်နိုင်ပြီး၊ `docker inspect` နဲ့ detailed config/status စစ်နိုင်ပါတယ်။  
`docker stats` နဲ့ resource usage monitor လုပ်နိုင်ပြီး၊ `docker cp`, `docker exec`, `docker diff`, `docker commit` နဲ့ container filesystem ကို manage လုပ်နိုင်ပါတယ်။  
Image ကို share/deploy လုပ်ချင်ရင် Docker Hub မှာ `login → tag → push` workflow ကိုသုံးရပါတယ်။  
မလိုတော့တဲ့ data တွေကို cleanup လုပ်တဲ့အခါ volumes/data မပျက်အောင် အမြဲသတိထားပါ။
