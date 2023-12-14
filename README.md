# Traefik + Let's Encrypt + vLLM + Docker Compose 

Quick guide in deploying production ready vLLM secure server behind Traefik reverse proxy and load balance vLLM when scaling. HTTPS let's encrypt certificate renewal is automated and comes with password protected Traefik dashboard.

## How to deploy on Public GPU Server with Real Domain Name

### Step 1. Install required dependencies

- Git
- Docker
- Docker Compose

Example installation on debian based systems:
```bash
sudo apt install git docker.io docker-compose
```

### Step 2. Clone the respoistory
```bash
git clone https://github.com/hurshd0/vllm-docker-traefik.git && cd vllm-docker-traefik
```

### Step 3. Create .env file
```bash
mv .env.example .env
nano .env
```

Fill in your environment variables
```bash
# Traefik settings
ADMIN_EMAIL=<admin@your-domain.com> # e.g. ADMIN_EMAIL=admin@website.com
DOMAIN=<yourwebsite.com> # e.g. DOMAIN=website.com
CERT_RESOLVER=letsencrypt # keep it blank for internal private or local net
TRAEFIK_USER=admin
TRAEFIK_PASSWORD_HASH=<your-password-hash> # e.g. TRAEFIK_PASSWORD_HASH=$2y$10$OfEBpHk52P/5Ad1qzDj79esMnuhaEbV5of7OBTSurzhtSENLeWzAW 

# vLLM settings
MODEL_NAME=<your-huggingface-hub-model> # e.g. MODEL_NAME="mistralai/Mistral-7B-Instruct-v0.1"
HF_TOKEN=<your-huggingface-hub-token> # e.g. HF_TOKEN="hf_XXXXXXXXXXXXXXXXX"
```
**Note**

- `TRAEFIK_PASSWORD_HASH` you will need Apache Utilities Package to generate admin password
> Install `sudo apt update && sudo apt install apache2-utils -y`

#### Set your own Traefik Dashboard admin password
```bash
htpasswd -nBC 10 admin
```

- `HF_TOKEN` get it from [Huggingface Hub](https://huggingface.co/docs/hub/security-tokens)

- You can leave `CERT_RESOLVER` empty if you test for local or internal deployment

### Step 4. Launch vLLM server
```
docker-compose up -d
```

Scale with:
```
docker-compose scale vllm=2
```