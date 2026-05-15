# Dockerfile, Image Build, Layers, CMD/ENTRYPOINT, ENV/ARG Notes

> ဒီ note က Dockerfile ရေးနည်း၊ image build လုပ်နည်း၊ image layers, CMD vs ENTRYPOINT, ENV vs ARG, Build → Tag → Push workflow တွေကို မြန်မာလို အတိုချုံးပြီး မှတ်ရလွယ်အောင် ရေးထားတာပါ။

---

## 1. Dockerfile ဆိုတာဘာလဲ?

`Dockerfile` ဆိုတာ Docker image တစ်ခုတည်ဆောက်ဖို့အတွက် instruction တွေကို အဆင့်လိုက်ရေးထားတဲ့ file ပါ။

ဥပမာ Python Flask app အတွက် Dockerfile—

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

ENV APP_NAME="Hello Docker World"

CMD ["python", "app.py"]
```

ဒီ Dockerfile က Python base image ကိုယူပြီး app dependencies install လုပ်၊ app code copy လုပ်ပြီး container start ဖြစ်တဲ့အခါ `python app.py` ကို run စေပါတယ်။

---

## 2. Dockerfile Instruction Breakdown

| Instruction | Meaning |
|---|---|
| `FROM` | Base image သတ်မှတ်ခြင်း |
| `WORKDIR` | Container ထဲမှာ အလုပ်လုပ်မည့် directory သတ်မှတ်ခြင်း |
| `COPY` | Local machine က file/folder ကို image ထဲ copy လုပ်ခြင်း |
| `RUN` | Image build time မှာ command run ခြင်း |
| `EXPOSE` | App က listen လုပ်မည့် port ကို document လုပ်ခြင်း |
| `ENV` | Runtime environment variable သတ်မှတ်ခြင်း |
| `CMD` | Container start ဖြစ်တဲ့အခါ default command run ခြင်း |

---

## 3. Image Build လုပ်ခြင်း

Dockerfile ရှိတဲ့ folder ထဲမှာ ဒီ command run ပါ—

```bash
docker build -t python-flask-app:1.0 .
```

အဓိပ္ပါယ်—

| Part | Meaning |
|---|---|
| `docker build` | Docker image build လုပ်မယ် |
| `-t` | Image name/tag ပေးမယ် |
| `python-flask-app:1.0` | Image name + version |
| `.` | Current directory ကို build context အဖြစ်သုံးမယ် |

Build အောင်မြင်ရင် image ကိုစစ်ရန်—

```bash
docker images
```

Sample output—

```bash
REPOSITORY          TAG     IMAGE ID       SIZE
python-flask-app    1.0     def456abc123   186MB
```

---

## 4. Build လုပ်တဲ့အခါ ဘာတွေဖြစ်လဲ?

Docker build လုပ်တဲ့အခါ—

1. Dockerfile ကိုဖတ်တယ်။
2. Instruction တစ်ကြောင်းချင်းစီကို run တယ်။
3. Dependencies install လုပ်တယ်။
4. App code copy လုပ်တယ်။
5. Final image တစ်ခုထုတ်ပေးတယ်။

Build command—

```bash
docker build -t python-flask-app:1.0 .
```

Sample build output အတိုချုံး—

```bash
[+] Building 8.7s
 => [1/5] FROM python:3.11-slim
 => [2/5] WORKDIR /app
 => [3/5] COPY requirements.txt .
 => [4/5] RUN pip install --no-cache-dir -r requirements.txt
 => [5/5] COPY app.py .
 => exporting to image
 => naming to docker.io/library/python-flask-app:1.0
```

---

## 5. Docker Image Layers

Docker image တွေက layer by layer တည်ဆောက်ထားတာပါ။ Dockerfile ထဲက instruction တစ်ခုချင်းစီက layer အသစ်တစ်ခုဖြစ်စေနိုင်ပါတယ်။

ဥပမာ—

```dockerfile
FROM python:3.11-slim       # Layer 1
WORKDIR /app                # Layer 2
COPY requirements.txt .     # Layer 3
RUN pip install -r requirements.txt  # Layer 4
COPY app.py .               # Layer 5
CMD ["python", "app.py"]    # Startup command
```

### Layer တွေရဲ့ အရေးကြီးချက်

- Unchanged layers တွေကို Docker က cache ပြန်သုံးနိုင်တယ်။
- Cache ပြန်သုံးနိုင်ရင် rebuild ပိုမြန်တယ်။
- Dependencies ကို code မတိုင်ခင် copy/install လုပ်ထားတာက best practice ပါ။

ကောင်းတဲ့ pattern—

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

မကောင်းတဲ့ pattern—

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

ဘာလို့လဲဆိုတော့ code file တစ်ခု ပြောင်းတိုင်း dependencies layer ကိုပါ rebuild လုပ်ရနိုင်လို့ပါ။

---

## 6. CMD vs ENTRYPOINT

`CMD` နဲ့ `ENTRYPOINT` နှစ်ခုလုံးက container start ဖြစ်တဲ့အခါ ဘာ run မလဲဆိုတာ သတ်မှတ်ပေးပါတယ်။ အဓိကကွာခြားချက်က runtime မှာ override လုပ်လို့ရ/မရ ဖြစ်ပါတယ်။

---

## 6.1 CMD

`CMD` က default command ဖြစ်ပါတယ်။ Runtime မှာ အလွယ်တကူ override လုပ်နိုင်ပါတယ်။

Dockerfile—

```dockerfile
CMD ["python", "app.py"]
```

Default run—

```bash
docker run python-flask-app
```

တကယ် run သွားတာ—

```bash
python app.py
```

CMD override လုပ်ချင်ရင်—

```bash
docker run -it python-flask-app /bin/bash
```

ဒီလို run လိုက်ရင် default `python app.py` မ run တော့ဘဲ `/bin/bash` ကို run သွားပါတယ်။

### CMD ကို ဘယ်အချိန်သုံးသင့်လဲ?

- Web app, API app တွေမှာ default command ပေးချင်တဲ့အခါ
- Debug လုပ်ဖို့ command override လုပ်ချင်တဲ့အခါ
- Container ထဲဝင်ပြီး inspect/debug လုပ်နိုင်ဖို့ flexibility လိုတဲ့အခါ

---

## 6.2 ENTRYPOINT

`ENTRYPOINT` က fixed executable ပုံစံနဲ့ သုံးပါတယ်။ Runtime argument တွေက ENTRYPOINT နောက်မှာ append ဖြစ်သွားပါတယ်။

Dockerfile—

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

Run—

```bash
docker run python-flask-app
```

တကယ် run သွားတာ—

```bash
python app.py
```

Argument ထပ်ပေးရင်—

```bash
docker run python-flask-app --help
```

တကယ် run သွားတာ—

```bash
python app.py --help
```

### ENTRYPOINT ကို ဘယ်အချိန်သုံးသင့်လဲ?

- CLI tool container တွေ
- Main executable ကို fixed ထားချင်တဲ့ container တွေ
- User ပေးတဲ့ argument တွေကို main command နောက်မှာ append လုပ်ချင်တဲ့အခါ

---

## 6.3 CMD vs ENTRYPOINT Table

| Feature | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command | Fixed executable |
| Override | အလွယ်တကူ override လုပ်နိုင် | မလွယ်ကူ |
| Runtime argument | Command ကို replace လုပ်နိုင် | Argument ကို append လုပ်တတ် |
| Best use case | Web apps, APIs, debug-friendly containers | CLI tools, fixed-behavior containers |
| Example | `CMD ["python", "app.py"]` | `ENTRYPOINT ["python", "app.py"]` |

**Rule of Thumb:**  
App ကို debug/override လုပ်ချင်ရင် `CMD` သုံးပါ။ Main executable မပြောင်းစေချင်ရင် `ENTRYPOINT` သုံးပါ။

---

## 7. ENV vs ARG

`ARG` နဲ့ `ENV` နှစ်ခုလုံးက value သိမ်းဖို့သုံးပေမယ့် သုံးတဲ့အချိန်မတူပါဘူး။

| Feature | ARG | ENV |
|---|---|---|
| Time | Build time | Runtime |
| Available inside running container | မရ | ရ |
| Override | `docker build --build-arg` | `docker run -e` |
| Dockerfile instruction | `ARG` | `ENV` |
| Best use case | Version, build option, build path | App config, port, environment name |

---

## 7.1 ARG Example

Dockerfile—

```dockerfile
ARG PYTHON_VERSION=3.11
FROM python:${PYTHON_VERSION}-slim

ARG APP_HOME=/app
WORKDIR ${APP_HOME}

COPY . ${APP_HOME}
```

Build command—

```bash
docker build \
  --build-arg PYTHON_VERSION=3.12 \
  --build-arg APP_HOME=/usr/src/app \
  -t myapp:1.0 .
```

`ARG` value တွေက image build လုပ်တဲ့အချိန်မှာပဲ အဓိကအသုံးဝင်ပါတယ်။ Running container ထဲမှာ application က access လုပ်ဖို့မဟုတ်ပါ။

---

## 7.2 ENV Example

Dockerfile—

```dockerfile
FROM python:3.11-slim

WORKDIR /app

ENV APP_NAME="Hello Docker World"
ENV APP_ENV=development
ENV PORT=5000

COPY . /app

CMD ["python", "app.py"]
```

Run time မှာ ENV override လုပ်ရန်—

```bash
docker run \
  -e APP_NAME="Production App" \
  -e APP_ENV=production \
  -e PORT=8080 \
  myapp:1.0
```

Container ထဲက ENV စစ်ရန်—

```bash
docker exec -it <container_id> printenv
```

Specific ENV ကြည့်ရန်—

```bash
docker exec -it <container_id> echo $APP_NAME
```

Python app ထဲက access လုပ်ချင်ရင်—

```python
import os

app_name = os.getenv("APP_NAME")
port = os.getenv("PORT", "5000")

print(app_name)
print(port)
```

---

## 8. CMD နဲ့ ENV တွဲသုံးတဲ့ ဥပမာ

Dockerfile မှာ `ENV` နဲ့ default value ပေးပြီး `CMD` က အဲ့ဒီ ENV value ကိုသုံးပြီး app ကို run စေနိုင်ပါတယ်။

### Important Note

JSON exec form ဖြစ်တဲ့ ဒီပုံစံမှာ ENV variable expansion မလုပ်ပါ—

```dockerfile
CMD ["python", "app.py", "--port", "$PORT"]
```

အပေါ်က `$PORT` က value ပြောင်းမသွားဘဲ literal string အဖြစ်နေသွားနိုင်ပါတယ်။

ENV variable ကို CMD ထဲမှာသုံးချင်ရင် shell form သို့မဟုတ် `sh -c` သုံးပါ။

---

### Correct Example: CMD + ENV

Dockerfile—

```dockerfile
FROM python:3.11-slim

WORKDIR /app

ENV APP_NAME="Docker Flask App"
ENV PORT=5000

COPY app.py .

CMD ["sh", "-c", "python app.py --host 0.0.0.0 --port ${PORT}"]
```

Default run—

```bash
docker run python-flask-app:1.0
```

တကယ် run သွားတာ—

```bash
python app.py --host 0.0.0.0 --port 5000
```

Runtime မှာ port ပြောင်းချင်ရင်—

```bash
docker run -e PORT=8080 python-flask-app:1.0
```

တကယ် run သွားတာ—

```bash
python app.py --host 0.0.0.0 --port 8080
```

### CMD + ENV ကို ဘယ်အချိန်သုံးမလဲ?

- App port ကို runtime မှာပြောင်းချင်တဲ့အခါ
- Environment-specific config တွေသုံးချင်တဲ့အခါ
- Same image ကို dev/staging/prod မှာ config ပြောင်းပြီး run ချင်တဲ့အခါ

---

## 9. Docker Hub Workflow: Build → Tag → Push

Docker Hub ကို image upload လုပ်ဖို့ flow က—

```text
Build Image → Tag Image → Push Image → Use Anywhere
```

---

## 9.1 Build Image

```bash
docker build -t python-flask-app:1.0 .
```

Local image တစ်ခု build ဖြစ်လာမယ်။

---

## 9.2 Tag Image

Docker Hub ကို push လုပ်ဖို့ image name ကို Docker Hub username/repository format နဲ့ tag ပေးရပါမယ်။

```bash
docker tag python-flask-app:1.0 yourdockerhubuser/python-flask-app:1.0
```

Format—

```text
username/repository:tag
```

ဥပမာ—

```text
aries/python-flask-app:1.0
```

---

## 9.3 Login to Docker Hub

```bash
docker login
```

Sample—

```bash
Username: yourdockerhubuser
Password: ********
Login Succeeded
```

---

## 9.4 Push Image

```bash
docker push yourdockerhubuser/python-flask-app:1.0
```

Push အောင်မြင်ရင် Docker Hub repository ထဲမှာ image tag ကိုမြင်ရပါမယ်။

---

## 9.5 Verify / Pull Image

တခြား machine ကနေ ပြန် pull လုပ်နိုင်ပါတယ်—

```bash
docker pull yourdockerhubuser/python-flask-app:1.0
```

Run လုပ်ရန်—

```bash
docker run -p 8080:5000 yourdockerhubuser/python-flask-app:1.0
```

---

## 10. Quick Reference

| Purpose | Command |
|---|---|
| Build image | `docker build -t myapp:1.0 .` |
| List images | `docker images` |
| Run image | `docker run myapp:1.0` |
| Run with port mapping | `docker run -p 8080:5000 myapp:1.0` |
| Run with ENV | `docker run -e PORT=8080 myapp:1.0` |
| Build with ARG | `docker build --build-arg PYTHON_VERSION=3.12 -t myapp:1.0 .` |
| Tag image | `docker tag myapp:1.0 username/myapp:1.0` |
| Login Docker Hub | `docker login` |
| Push image | `docker push username/myapp:1.0` |
| Pull image | `docker pull username/myapp:1.0` |
| Check ENV inside container | `docker exec -it <id> printenv` |

---

## 11. Best Practices

- Dockerfile ကို simple, readable ဖြစ်အောင်ရေးပါ။
- Small base image သုံးပါ။ ဥပမာ `python:3.11-slim`
- Dependencies ကို app code မတိုင်ခင် install လုပ်ပါ။
- `.dockerignore` သုံးပြီး မလိုတဲ့ files တွေ build context ထဲမပါအောင်လုပ်ပါ။
- `CMD` ကို app default command အတွက်သုံးပါ။
- `ENTRYPOINT` ကို fixed executable/CLI tool အတွက်သုံးပါ။
- Build-time values အတွက် `ARG` သုံးပါ။
- Runtime config အတွက် `ENV` သုံးပါ။
- Docker Hub push မလုပ်ခင် `docker login` လုပ်ထားပါ။
- Meaningful tags သုံးပါ။ ဥပမာ `v1.0`, `v2.0.1`, `latest`

---

## Final Key Takeaway

Dockerfile က image တည်ဆောက်ဖို့ recipe ဖြစ်ပြီး Docker image က layer တွေနဲ့တည်ဆောက်ထားပါတယ်။  
`ARG` က build time အတွက်၊ `ENV` က runtime အတွက်ဖြစ်ပါတယ်။  
`CMD` က default command ဖြစ်ပြီး override လုပ်ရလွယ်ပါတယ်။ `ENTRYPOINT` က fixed executable အတွက်ပိုသင့်ပါတယ်။  
Real workflow မှာ image ကို build လုပ်၊ Docker Hub format နဲ့ tag ပေး၊ login လုပ်ပြီး push လုပ်နိုင်ရပါမယ်။
