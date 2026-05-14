# Docker Basic Commands Notes
## Container Run, Manage, Logs, Cleanup

## 1. Docker Installation စစ်ခြင်း

Docker install ဖြစ်/မဖြစ် version စစ်ရန်—

```bash
$ docker --version
Docker version 24.0.6, build ed223bc
```

Docker Engine နဲ့ CLI ချိတ်ဆက်မှု အလုပ်လုပ်/မလုပ် စစ်ရန်—

```bash
$ docker info
Client:
 Version:    24.0.6
 Context:    default
 Debug Mode: false

Server:
 Containers: 0
 Images: 0
 Server Version: 24.0.6
 Storage Driver: overlay2
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
```

**မှတ်ရန်**  
`docker --version` က Docker version စစ်တာပါ။  
`docker info` က Docker Engine running ဖြစ်လား၊ CLI က Engine ကိုချိတ်နိုင်လား စစ်တာပါ။

---

## 2. First Container Run လုပ်ခြင်း

ပထမဆုံး test image ဖြစ်တဲ့ `hello-world` ကို pull လုပ်ရန်—

```bash
$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

Container run လုပ်ရန်—

```bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

ဒီ output ထွက်လာရင် Docker installation အလုပ်လုပ်နေပြီ။

### Image vs Container

| Term | Meaning |
|---|---|
| Image | Container အတွက် blueprint/template |
| Container | Image ကနေ run ဖြစ်လာတဲ့ running instance |

---

## 3. Container Run Modes

## 3.1 Interactive Mode `-it`

Ubuntu container ကို interactive terminal နဲ့ run လုပ်ရန်—

```bash
$ docker run -it ubuntu bash
root@container:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  tmp  usr  var
root@container:/# exit
exit
```

`-it` ဆိုတာ container ထဲကို terminal နဲ့ဝင်ပြီး command တွေ run လို့ရအောင်လုပ်တာပါ။

- `-i` = interactive mode
- `-t` = terminal/TTY ပေးခြင်း

---

## 3.2 Detached Mode `-d`

Nginx container ကို background မှာ run လုပ်ရန်—

```bash
$ docker run -d -p 8080:80 --name web-server nginx
a1b2c3d4e5f6g7h8i9j0k112m3n4o5p6
```

ဒီ command က Nginx web server ကို background မှာ run စေပါတယ်။

Port mapping က—

```text
8080:80
```

အဓိပ္ပါယ်က—

```text
LOCAL_PORT:CONTAINER_PORT
```

Browser မှာ ဒီလိုဖွင့်နိုင်ပါတယ်—

```text
http://localhost:8080
```

`localhost:8080` ကိုသွားရင် request က container ထဲက port `80` ကို forward လုပ်သွားပါတယ်။

---

## 3.3 Auto Remove Mode `--rm`

Temporary container run ပြီးပြီးချင်း auto delete လုပ်ရန်—

```bash
$ docker run --rm ubuntu echo "Hello DevOps"
Hello DevOps
```

`--rm` က container stop ဖြစ်တာနဲ့ automatically remove လုပ်ပေးပါတယ်။

---

## 4. Running Containers ကြည့်ခြင်း

လက်ရှိ run နေတဲ့ containers တွေကြည့်ရန်—

```bash
$ docker ps
CONTAINER ID   IMAGE   STATUS      PORTS                  NAMES
a1b2c3d4e5f6   nginx   Up 2 mins   0.0.0.0:8080->80/tcp   web-server
```

`docker ps` က **running containers only** ကိုပဲပြပါတယ်။

Output ထဲက—

```text
0.0.0.0:8080->80/tcp
```

ဆိုတာ host machine port `8080` က container port `80` ကိုချိတ်ထားတာပါ။

---

## 5. All Containers ကြည့်ခြင်း

Running + stopped containers အားလုံးကြည့်ရန်—

```bash
$ docker ps -a
CONTAINER ID   IMAGE         STATUS                   PORTS                  NAMES
a1b2c3d4e5f6   nginx         Up 2 mins                0.0.0.0:8080->80/tcp   web-server
x9y8z7w6v5u4   hello-world   Exited (0) 10 mins ago                          test-container
```

| Status | Meaning |
|---|---|
| Up | Container run နေသည် |
| Exited | Container stop ဖြစ်သွားသည် |

`docker ps` မှာမမြင်ရတဲ့ stopped container တွေကို `docker ps -a` နဲ့မြင်နိုင်ပါတယ်။

---

## 6. Container Logs ကြည့်ခြင်း

Container output/log ကြည့်ရန်—

```bash
$ docker logs test-container
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Follow logs လုပ်ရန်—

```bash
$ docker logs -f web-server
```

Last 20 lines ကြည့်ရန်—

```bash
$ docker logs --tail 20 web-server
```

**မှတ်ရန်**  
Application မအလုပ်လုပ်ရင် ပထမဆုံး `docker logs` နဲ့ error ကိုစစ်သင့်ပါတယ်။

---

## 7. Container Lifecycle Control

## 7.1 Stop Container

Running container ကို stop လုပ်ရန်—

```bash
$ docker stop web-server
web-server
```

`docker stop` က running container ကို graceful stop လုပ်ပေးပါတယ်။

---

## 7.2 Start Container

Stopped container ကိုပြန် start လုပ်ရန်—

```bash
$ docker start web-server
web-server
```

ဒီ command က container အဟောင်းကိုပဲပြန် run တာပါ။

---

## 7.3 Restart Container

Container ကို restart လုပ်ရန်—

```bash
$ docker restart web-server
web-server
```

Restart ဆိုတာ stop လုပ်ပြီး ပြန် start လုပ်တာပါ။

---

## 7.4 Remove Container

Container ကို remove/delete လုပ်ရန်—

```bash
$ docker rm web-server
web-server
```

Running ဖြစ်နေသေးရင် အရင် stop လုပ်ရပါမယ်—

```bash
$ docker stop web-server
web-server

$ docker rm web-server
web-server
```

**သတိထားရန်**  
`docker rm` က container ကို permanently delete လုပ်တာဖြစ်လို့ မလိုတော့မှ run ပါ။

---

## 8. Running Container ထဲဝင်ခြင်း `docker exec -it`

Run နေတဲ့ container ထဲကိုဝင်ရန်—

```bash
$ docker exec -it web-server /bin/bash
root@web-server:/# ls
bin  boot  dev  etc  home  lib  media  opt  proc  root  tmp  usr  var
```

OS info စစ်ရန်—

```bash
root@web-server:/# cat /etc/os-release
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
ID=debian
```

Container ထဲကနေထွက်ရန်—

```bash
root@web-server:/# exit
exit
```

`/bin/bash` မရှိရင် `sh` သုံးနိုင်ပါတယ်—

```bash
$ docker exec -it web-server sh
```

**မှတ်ရန်**  
`docker exec` က new container မဖန်တီးပါဘူး။ Existing running container ထဲကိုဝင်တာပါ။ Container က running ဖြစ်နေမှသာ အသုံးပြုလို့ရပါတယ်။

---

## 9. Basic Cleanup

## 9.1 Specific Container Remove

မလိုတော့တဲ့ container တစ်ခု remove လုပ်ရန်—

```bash
$ docker rm web-server
web-server
```

---

## 9.2 Remove All Stopped Containers

Stopped containers အားလုံး remove လုပ်ရန်—

```bash
$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
a1b2c3d4e5f6
x9y8z7w6v5u4
```

Running containers တွေကိုတော့ မဖျက်ပါဘူး။ Stopped containers တွေကိုပဲဖျက်ပါတယ်။

---

## 9.3 Remove Unused Docker Data

Unused containers, networks, dangling images, build cache တွေကိုရှင်းရန်—

```bash
$ docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all build cache

Are you sure you want to continue? [y/N] y
Total reclaimed space: 2.34GB
```

Unused images တွေပါပိုရှင်းချင်ရင်—

```bash
$ docker system prune -a
```

`-a` က image တွေပါပိုဖျက်နိုင်လို့ သတိထားသုံးပါ။

---

## Workflow Summary

Docker container workflow ကို အတိုချုံးမှတ်ရင်—

```text
Pull image → Run container → Check status → View logs → Manage lifecycle → Cleanup
```

Container ကို run ပြီးနောက် `docker ps` နဲ့ status စစ်နိုင်ပြီး၊ error ရှိရင် `docker logs` နဲ့ကြည့်နိုင်ပါတယ်။ Container ထဲဝင်စစ်ချင်ရင် `docker exec -it` ကိုသုံးပြီး၊ မလိုတော့ရင် `docker stop`, `docker rm`, `docker container prune` နဲ့ cleanup လုပ်နိုင်ပါတယ်။

---

## Quick Command Cheat Sheet

| Purpose | Command |
|---|---|
| Check Docker version | `docker --version` |
| Check Docker Engine info | `docker info` |
| Pull image | `docker pull hello-world` |
| Run container | `docker run hello-world` |
| Run interactive container | `docker run -it ubuntu bash` |
| Run background container | `docker run -d -p 8080:80 nginx` |
| Run with name | `docker run --name web-server nginx` |
| Auto remove container | `docker run --rm ubuntu echo "Hello"` |
| View running containers | `docker ps` |
| View all containers | `docker ps -a` |
| View logs | `docker logs <name>` |
| Follow logs | `docker logs -f <name>` |
| View last logs | `docker logs --tail 20 <name>` |
| Enter running container | `docker exec -it <name> /bin/bash` |
| Enter with sh | `docker exec -it <name> sh` |
| Stop container | `docker stop <name>` |
| Start container | `docker start <name>` |
| Restart container | `docker restart <name>` |
| Remove container | `docker rm <name>` |
| Remove stopped containers | `docker container prune` |
| Remove unused data | `docker system prune` |

---

## Final Key Takeaway

Docker မှာ **image** က blueprint ဖြစ်ပြီး **container** က running instance ဖြစ်ပါတယ်။  
Container ကို run, check, debug, manage, logs ကြည့်, exec နဲ့ဝင်စစ်, cleanup လုပ်နိုင်ရင် Docker basic workflow ကိုနားလည်ပြီလို့ပြောနိုင်ပါတယ်။
