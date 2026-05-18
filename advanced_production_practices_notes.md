# Docker Day 6 — Advanced Production Practices Notes 

## အကျဉ်းချုပ်

Docker Day 6 မှာ လေ့လာရမယ့်အကြောင်းအရာက **Production မှာသုံးလို့ရတဲ့ Docker Best Practices** တွေပါ။

Container တစ်ခုကို run လိုက်ရုံနဲ့ Production-ready ဖြစ်သွားတာမဟုတ်ပါဘူး။  
Production မှာ သုံးမယ်ဆိုရင် image size သေးရမယ်၊ security ကောင်းရမယ်၊ resource usage ကို control လုပ်နိုင်ရမယ်၊ health check ပါရမယ်၊ secrets တွေကိုလည်း မပါအောင်ကာကွယ်ရပါမယ်။

ဒီ note ထဲမှာ ပါဝင်မယ့်အကြောင်းအရာတွေက—

1. Multi-stage Builds
2. Container ကို Non-root User နဲ့ Run ခြင်း
3. CPU / RAM Resource Limits
4. Vulnerability Scanning
5. `.dockerignore` နဲ့ Secrets ကာကွယ်ခြင်း
6. Docker `HEALTHCHECK`
7. Docker Buildx

---

# 1. Multi-stage Builds

## Multi-stage Build ဆိုတာဘာလဲ?

**Multi-stage build** ဆိုတာ Dockerfile တစ်ခုထဲမှာ `FROM` instruction ကို တစ်ကြိမ်ထက်ပိုသုံးပြီး image build လုပ်တဲ့နည်းပါ။

အဓိက idea က—

- ပထမ stage မှာ application ကို build လုပ်မယ်
- ဒုတိယ stage မှာ build ပြီးသား output ကိုပဲ production image ထဲထည့်မယ်

ဒီနည်းကြောင့် build tools တွေ၊ source files အကုန်လုံး၊ မလိုအပ်တဲ့ dependencies တွေ final image ထဲ မပါတော့ပါဘူး။

---

## Multi-stage Build သုံးရတဲ့အကြောင်းရင်း

Multi-stage build သုံးရင်—

- Image size သေးသွားတယ်
- Deploy လုပ်တာ ပိုမြန်တယ်
- Security ပိုကောင်းတယ်
- Production image ပိုရှင်းတယ်
- Build tools တွေ final image ထဲ မပါဘူး
- Attack surface လျော့သွားတယ်

---

## Example: Node.js / Next.js Multi-stage Dockerfile

```dockerfile
# Stage 1: Build stage
FROM node:22-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm install

COPY . .
RUN npm run build

# Stage 2: Production stage
FROM node:22-alpine AS runner

WORKDIR /app

COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000

CMD ["npm", "start"]
```

### ရှင်းပြချက်

`builder` stage မှာ application ကို build လုပ်ပါတယ်။  
`runner` stage မှာတော့ production မှာ run ဖို့လိုတဲ့ files တွေကိုပဲ copy လုပ်ပါတယ်။

`COPY --from=builder` ဆိုတာ ပထမ stage ထဲက file တွေကို ဒုတိယ stage ထဲကို copy လုပ်တာပါ။

---

## Example: Vue.js App ကို Nginx နဲ့ Serve လုပ်ခြင်း

```dockerfile
# Stage 1: Build Vue app
FROM node:22-alpine AS build

WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm install

COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### ရှင်းပြချက်

ဒီ example မှာ—

- ပထမ stage မှာ Vue app ကို build လုပ်တယ်
- Build result က `/app/dist` ထဲထွက်တယ်
- ဒုတိယ stage မှာ `nginx:alpine` ကိုသုံးပြီး static files တွေကို serve လုပ်တယ်

Final image ထဲမှာ Node.js build tools မပါတော့ဘဲ Nginx နဲ့ static files ပဲ ပါသွားပါတယ်။

---

# 2. Container ကို Non-root User နဲ့ Run ခြင်း

## ဘာကြောင့် Root နဲ့ မ Run သင့်တာလဲ?

Docker container အများစုက default အနေနဲ့ `root` user နဲ့ run တတ်ပါတယ်။  
ဒါက security အရ မကောင်းပါဘူး။

Application ထဲမှာ vulnerability ရှိပြီး attacker က container ထဲဝင်နိုင်သွားရင် root permission ရနိုင်ပါတယ်။

ဒါကြောင့် production မှာ container ကို **non-root user** နဲ့ run သင့်ပါတယ်။

---

## Principle of Least Privilege

**Principle of Least Privilege** ဆိုတာ application တစ်ခု run ဖို့လိုအပ်တဲ့ permission အနည်းဆုံးပဲပေးတဲ့ security principle ပါ။

လိုအပ်တာထက်ပိုတဲ့ permission မပေးတာက security ကိုပိုကောင်းစေပါတယ်။

---

## Root နဲ့ Run တာ

Root နဲ့ run ရင်—

- Container ထဲမှာ full root access ရနိုင်တယ်
- System files တွေ modify လုပ်နိုင်တယ်
- Sensitive data တွေ access လုပ်နိုင်တယ်
- Security risk ပိုမြင့်တယ်

---

## Non-root နဲ့ Run တာ

Non-root user နဲ့ run ရင်—

- Permission ကန့်သတ်ထားတယ်
- Container compromise ဖြစ်ရင်တောင် damage လျော့တယ်
- System files တွေ modify လုပ်ဖို့ခက်တယ်
- Production အတွက် ပိုသင့်တော်တယ်

---

## Example: Node.js App ကို Non-root User နဲ့ Run ခြင်း

```dockerfile
FROM node:22-alpine

# Non-root group and user create လုပ်ခြင်း
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY . .

# Application files ownership ပြောင်းခြင်း
RUN chown -R appuser:appgroup /app

# Non-root user သို့ switch လုပ်ခြင်း
USER appuser

EXPOSE 3000

CMD ["node", "server.js"]
```

---

## Container ထဲက User ကို စစ်ခြင်း

```bash
docker exec -it <container_name> whoami
```

Expected output:

```text
appuser
```

User ID ကိုစစ်မယ်ဆိုရင်—

```bash
docker exec -it <container_name> id
```

Example output:

```text
uid=1000(appuser) gid=1000(appgroup)
```

---

# 3. Resource Limits: CPU / RAM

## Resource Limits ဆိုတာဘာလဲ?

Resource limits ဆိုတာ container တစ်ခုက သုံးနိုင်တဲ့ CPU နဲ့ RAM ပမာဏကို ကန့်သတ်တာပါ။

Limit မထားရင် container တစ်ခုက host machine ရဲ့ resource အကုန်နီးပါးသုံးပြီး တခြား container တွေကို ထိခိုက်စေနိုင်ပါတယ်။

---

## Resource Limits ဘာကြောင့်အရေးကြီးလဲ?

Resource limits ထားခြင်းက—

- Container တစ်ခုကြောင့် host တစ်ခုလုံး down မသွားအောင်ကာကွယ်တယ်
- Out of Memory error တွေ လျော့စေတယ်
- Application stability ပိုကောင်းစေတယ်
- Multi-tenant environment မှာ resource sharing ပိုမျှတစေတယ်
- Production workload တွေအတွက် ပိုယုံကြည်စိတ်ချရတယ်

---

## Common Resource Flags

| Flag | အဓိပ္ပါယ် | Example |
|---|---|---|
| `--memory` | Container သုံးနိုင်တဲ့ maximum RAM | `--memory="512m"` |
| `--memory-swap` | RAM + Swap total limit | `--memory-swap="1g"` |
| `--cpus` | CPU core limit | `--cpus="1.0"` |
| `--pids-limit` | Process အရေအတွက် limit | `--pids-limit=100` |
| `--blkio-weight` | Disk I/O weight | `--blkio-weight=500` |

---

## Example: CPU/RAM Limit နဲ့ Container Run ခြင်း

```bash
docker run -d \
  --name myapp \
  --memory="512m" \
  --memory-swap="1g" \
  --cpus="1.0" \
  --pids-limit=100 \
  -p 8080:3000 \
  myapp:latest
```

### ရှင်းပြချက်

ဒီ command မှာ—

- `--memory="512m"` က RAM ကို 512MB limit ထားတာ
- `--memory-swap="1g"` က RAM + swap ကို 1GB limit ထားတာ
- `--cpus="1.0"` က CPU 1 core စာ limit ထားတာ
- `--pids-limit=100` က process 100 ထိပဲ run ခွင့်ပေးတာပါ

---

## Resource Usage ကို Monitor လုပ်ခြင်း

```bash
docker stats
```

ဒီ command က running containers တွေရဲ့ CPU, memory, network, disk usage တွေကို live ပြပါတယ်။

---

# 4. Vulnerability Scanning

## Vulnerability Scanning ဆိုတာဘာလဲ?

Vulnerability scanning ဆိုတာ Docker image ထဲမှာ security weakness တွေရှိမရှိ စစ်တာပါ။

ဥပမာ—

- Vulnerable packages
- Old libraries
- Known CVEs
- Insecure dependencies

Production ကို push မလုပ်ခင် scan လုပ်သင့်ပါတယ်။

---

## Vulnerability Scanning ဘာကြောင့်အရေးကြီးလဲ?

Vulnerability scan လုပ်ခြင်းက—

- Attacker မ exploit လုပ်ခင် issue တွေကို သိနိုင်တယ်
- Attack surface လျော့စေတယ်
- Compliance requirements တွေကို ကူညီပေးတယ်
- Secure image တည်ဆောက်နိုင်တယ်

---

## Popular Scanning Tools

## Trivy

Trivy က open-source vulnerability scanner တစ်ခုပါ။

Trivy နဲ့ scan လုပ်နိုင်တာတွေက—

- Docker images
- Filesystem
- Git repository
- Infrastructure as Code files

---

## Docker Scout

Docker Scout က Docker ecosystem ထဲမှာ integrate ဖြစ်တဲ့ scanning tool ပါ။

Docker Scout နဲ့—

- Image vulnerabilities တွေကြည့်နိုင်တယ်
- Remediation advice ရနိုင်တယ်
- Docker Hub နဲ့ integrate လုပ်နိုင်တယ်
- Continuous scanning လုပ်နိုင်တယ်

---

## Trivy နဲ့ Image Scan လုပ်ခြင်း

Basic image scan:

```bash
trivy image myapp:latest
```

High နဲ့ Critical vulnerabilities ပဲကြည့်ချင်ရင်—

```bash
trivy image --severity HIGH,CRITICAL myapp:latest
```

Unfixed vulnerabilities တွေကို ignore လုပ်ချင်ရင်—

```bash
trivy image --ignore-unfixed myapp:latest
```

Table format နဲ့ output ထုတ်ချင်ရင်—

```bash
trivy image -f table myapp:latest
```

---

## Docker Scout Commands

Quick view:

```bash
docker scout quickview
```

Detailed CVE report:

```bash
docker scout cves myapp:latest
```

---

## Severity Levels

| Level | အဓိပ္ပါယ် |
|---|---|
| Critical | အလွန်အရေးကြီး။ ချက်ချင်း fix လုပ်သင့် |
| High | Exploit ဖြစ်နိုင်ခြေမြင့်။ မြန်မြန် fix လုပ်သင့် |
| Medium | Risk ရှိနိုင်။ နောက် update cycle မှာ fix လုပ်သင့် |
| Low | Impact နည်း။ အဆင်ပြေချိန် fix လုပ်နိုင် |
| Unknown | Risk အတိအကျမသိနိုင်သေး |

---

## Vulnerability Scanning Best Practices

- CI/CD pipeline ထဲမှာ scan ထည့်ပါ
- HIGH / CRITICAL တွေရှိရင် build fail လုပ်ပါ
- Minimal base images သုံးပါ
- Base image နဲ့ dependencies တွေ update လုပ်ပါ
- Image ကို regular rescan လုပ်ပါ

---

# 5. `.dockerignore` နဲ့ Secrets ကာကွယ်ခြင်း

## `.dockerignore` ဆိုတာဘာလဲ?

`.dockerignore` ဆိုတာ Docker build context ထဲမဝင်စေချင်တဲ့ files/folders တွေကို သတ်မှတ်တဲ့ file ပါ။

`.gitignore` နဲ့ဆင်တူပေမယ့် မတူပါဘူး။

Important:

- `.gitignore` က Git အတွက်ပဲ အကျိုးသက်ရောက်တယ်
- `.dockerignore` က Docker build context အတွက် အကျိုးသက်ရောက်တယ်

---

## ဘာကြောင့် `.dockerignore` လိုအပ်လဲ?

Docker build လုပ်တဲ့အခါ project folder ထဲက files တွေ Docker daemon ဆီကို ပို့ပါတယ်။  
`.dockerignore` မထားရင် sensitive files တွေ Docker build context ထဲ ပါသွားနိုင်ပါတယ်။

မပါသင့်တဲ့ files/folders တွေ—

- `.env`
- `.git`
- `node_modules`
- `secrets/`
- `*.pem`
- log files
- cache files

---

## Example `.dockerignore`

```dockerignore
.env
.git
node_modules
*.pem
secrets/
```

---

## Example Project Structure

```text
project/
├── .env
├── .git/
├── node_modules/
├── secrets/
├── package.json
├── app.js
├── src/
└── Dockerfile
```

ဒီ structure မှာ `.env`, `.git`, `node_modules`, `secrets/` တွေကို Docker build context ထဲ မပါအောင် `.dockerignore` ထဲထည့်ထားသင့်ပါတယ်။

---

## `.env` ကို Image ထဲမထည့်ဘဲ Runtime မှာသုံးခြင်း

`.env` file ကို image ထဲ copy မလုပ်သင့်ပါဘူး။  
Container run တဲ့အချိန်မှာ `--env-file` နဲ့ pass လုပ်တာ ပိုသင့်တော်ပါတယ်။

```bash
docker run -d \
  --env-file .env \
  -p 3000:3000 \
  --name myapp \
  myapp:v1
```

---

## Docker Compose နဲ့ `.env` သုံးခြင်း

```yaml
services:
  myapp:
    build: .
    env_file:
      - .env
    ports:
      - "3000:3000"
```

---

# 6. Docker HEALTHCHECK

## HEALTHCHECK ဆိုတာဘာလဲ?

`HEALTHCHECK` ဆိုတာ container ထဲက application အလုပ်လုပ်နေသေးလား စစ်ပေးတဲ့ Docker feature ပါ။

Container process က run နေတယ်ဆိုရင်တောင် application က response မပြန်နိုင်တာမျိုး ဖြစ်နိုင်ပါတယ်။  
ဒါကြောင့် health check ထည့်ထားရင် container state ကိုပိုပြီးမှန်မှန်ကန်ကန် သိနိုင်ပါတယ်။

---

## HEALTHCHECK States

| State | အဓိပ္ပါယ် |
|---|---|
| `starting` | Container စတင်နေဆဲ |
| `healthy` | Container ကောင်းကောင်းအလုပ်လုပ်နေ |
| `unhealthy` | Health check fail ဖြစ်နေ |

---

## HEALTHCHECK Options

| Option | အဓိပ္ပါယ် |
|---|---|
| `--interval` | Check တစ်ကြိမ်နဲ့တစ်ကြိမ်ကြားအချိန် |
| `--timeout` | Response စောင့်မယ့် maximum time |
| `--retries` | Fail ဘယ်နှစ်ကြိမ်ဖြစ်မှ unhealthy လုပ်မလဲ |
| `--start-period` | Container စတင်ပြီး check မလုပ်ခင် grace period |

---

## Example Dockerfile with HEALTHCHECK

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package.json .
RUN npm install --production

COPY . .

HEALTHCHECK --interval=30s \
  --timeout=5s \
  --retries=3 \
  --start-period=10s \
  CMD curl --fail http://localhost:3000 || exit 1

CMD ["node", "server.js"]
```

---

## Health Check Examples

HTTP endpoint စစ်ခြင်း—

```bash
curl --fail http://localhost:3000
```

Database ready ဖြစ်မဖြစ် စစ်ခြင်း—

```bash
pg_isready -U user -d db
```

File ရှိမရှိ စစ်ခြင်း—

```bash
test -f /tmp/healthy
```

Port open ဖြစ်မဖြစ် စစ်ခြင်း—

```bash
nc -z localhost 5432
```

---

## Container Health ကို ကြည့်ခြင်း

```bash
docker ps
```

ပိုပြီးအသေးစိတ်ကြည့်ချင်ရင်—

```bash
docker inspect <container_name>
```

---

# 7. Restart Policies

## Restart Policy ဆိုတာဘာလဲ?

Restart policy ဆိုတာ container stop ဖြစ်သွားတဲ့အခါ Docker က ပြန် run ပေးမလားဆိုတာ သတ်မှတ်တာပါ။

Production မှာ service တွေ automatically recover ဖြစ်ဖို့အတွက် restart policy သုံးသင့်ပါတယ်။

---

## Restart Policy Types

| Policy | Behavior | သုံးသင့်တဲ့နေရာ |
|---|---|---|
| `no` | ဘယ်တော့မှ auto restart မလုပ် | Debugging / one-off tasks |
| `on-failure[:max-retries]` | Error နဲ့ exit ဖြစ်မှ restart | Batch jobs / worker jobs |
| `always` | အမြဲ restart လုပ် | Critical service |
| `unless-stopped` | ကိုယ်တိုင် stop မလုပ်သရွေ့ restart | Production service အများစု |

---

## Run Container with Restart Policy

```bash
docker run -d \
  --restart unless-stopped \
  --name myapp \
  myapp:v1
```

---

## Docker Compose Restart Policy

```yaml
services:
  myapp:
    image: myapp:v1
    restart: unless-stopped
```

---

# 8. Docker Buildx

## Docker Buildx ဆိုတာဘာလဲ?

Docker Buildx ဆိုတာ advanced Docker image build tool ပါ။  
အထူးသဖြင့် **multi-platform image** တွေ build လုပ်ရာမှာသုံးပါတယ်။

Buildx နဲ့ architecture မတူတဲ့ platform တွေအတွက် image တစ်ခု build လုပ်နိုင်ပါတယ်။

ဥပမာ—

- `linux/amd64`
- `linux/arm64`

---

## Buildx ဘာကြောင့်သုံးလဲ?

Buildx သုံးရင်—

- Multiple architectures အတွက် build လုပ်နိုင်တယ်
- Apple Silicon Mac, Intel Mac, Linux server, Cloud environment တွေအတွက် support ကောင်းတယ်
- CI/CD pipeline မှာအသုံးဝင်တယ်
- Build caching နဲ့ faster build ရနိုင်တယ်
- Multi-platform image ကို registry ထဲ push လုပ်နိုင်တယ်

---

## Buildx Builder Create လုပ်ခြင်း

```bash
docker buildx create --use
```

Builder ကိုစစ်ခြင်း—

```bash
docker buildx inspect --bootstrap
```

---

## Multi-platform Image Build လုပ်ခြင်း

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:v1 \
  --push .
```

### သတိပြုရန်

Multi-platform image build လုပ်တဲ့အခါ `--push` ကိုသုံးတာများပါတယ်။  
Local Docker image store ထဲကို multi-platform image တစ်ခုလုံး load လုပ်တာထက် registry ကို push လုပ်တာ ပိုသင့်တော်ပါတယ်။

---

## Image Platform တွေ Verify လုပ်ခြင်း

```bash
docker buildx imagetools inspect myapp:v1
```

Example output:

```text
Platforms:
  - linux/amd64
  - linux/arm64
```

---

# Production Checklist

Production မှာ Docker container သုံးမယ်ဆိုရင် အောက်ပါအချက်တွေ စစ်ပါ။

- Multi-stage build သုံးထားလား
- Base image သေးသေးသုံးထားလား
- Container ကို root မဟုတ်တဲ့ user နဲ့ run ထားလား
- CPU / RAM limits ထားထားလား
- `.dockerignore` ထည့်ထားလား
- Secrets တွေ image ထဲမပါအောင်ကာကွယ်ထားလား
- Image vulnerability scan လုပ်ထားလား
- `HEALTHCHECK` ထည့်ထားလား
- Restart policy ထည့်ထားလား
- Container resource usage ကို monitor လုပ်ထားလား
- Multi-platform build လိုအပ်ရင် Buildx သုံးထားလား

---

# Important Commands Summary

## Image build လုပ်ခြင်း

```bash
docker build -t myapp:v1 .
```

## Container run ခြင်း

```bash
docker run -d --name myapp -p 3000:3000 myapp:v1
```

## Non-root user သုံးခြင်း

```dockerfile
USER appuser
```

## Resource limit ထည့်ခြင်း

```bash
docker run -d --memory="512m" --cpus="1.0" myapp:v1
```

## Resource usage ကြည့်ခြင်း

```bash
docker stats
```

## Image scan လုပ်ခြင်း

```bash
trivy image myapp:v1
```

## `.env` file ကို runtime မှာသုံးခြင်း

```bash
docker run --env-file .env myapp:v1
```

## Restart policy ထည့်ခြင်း

```bash
docker run -d --restart unless-stopped myapp:v1
```

## Buildx multi-platform image build

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:v1 --push .
```

---

# Final Summary


Production-ready Docker container ဖြစ်ဖို့ဆိုရင်—

- Image size သေးရမယ်
- Security ကောင်းရမယ်
- Root user နဲ့ မ run သင့်ဘူး
- Resource usage ကို limit ထားရမယ်
- Secrets တွေ image ထဲမပါရဘူး
- Vulnerability scan လုပ်ထားရမယ်
- Health check ပါရမယ်
- Restart policy ပါရမယ်
- Monitoring လုပ်နိုင်ရမယ်

ဒီအချက်တွေကိုလိုက်နာရင် Docker application တွေကို ပိုပြီး secure, stable, lightweight, reliable ဖြစ်အောင် production မှာ run နိုင်ပါတယ်။
