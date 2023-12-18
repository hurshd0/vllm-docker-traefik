# vLLM + Docker + Traefik 


<img src="https://i.imgur.com/ZBQX3DW.jpeg" alt="cloud containerized app"  width="45%"/>
<hr/>

Quick guide in deploying production ready Mistral-7B or Mixtral-8x7B vLLM secure server behind Traefik reverse proxy and load balancer. By default I am using HTTPS let's encrypt certificate, has automated renewal and comes with password protected Traefik dashboard.

> __Note__  
>
> - You can disable Traefik dashboard for keep it internal access only
> - You can substitute Let's Encrypt with your own or CloudFlare or third-party for DNS challenge
> - Add, Crowdsec or WAF like Cloudflare to secure your DNS Zone and any bot, or Cybersecurity threats   

## How to deploy on Public GPU Server with Real Domain Name

### Step 1. Install required dependencies

- Git
- Docker ~ v24
- Docker Compose ~ v2

**Check below guides in installing Docker and Docker Compose on Ubuntu 20.04**

- [Install Docker on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)
- [Install Docker Compose on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)

> Note: For NVIDIA GPU you will need docker compose version > v1.28.0, otherwise you will get error

### Step 2. Clone the respoistory
```bash
git clone https://github.com/hurshd0/vllm-docker-traefik.git && cd vllm-docker-traefik
```

### Step 3. Create .env file 
```bash
mv .env.example .env
nano .env
```

**Fill in your environment variables**
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

#### [Optional] Set your own Traefik Dashboard admin password

> **Note:**
> - Install `sudo apt update && sudo apt install apache2-utils -y`

```bash
htpasswd -nBC 10 admin
```
#### Get Huggingface Hub Token
- `HF_TOKEN` get it from [Huggingface Hub](https://huggingface.co/docs/hub/security-tokens)

- You can leave `CERT_RESOLVER` empty if you want to test for local deployment
```bash
CERT_RESOLVER=
```

### Step 4. Configure vLLM Server for SINGLE or MULTI-GPU setup

#### A. For SINGLE GPU setup 

Start vLLM server

```bash
docker compose up -d
```

Check logs
```bash
docker compose logs
```

#### B. For MULTI-GPU setup serving Mixtral-8x7B 

Get NVIDIA Device IDs
```bash
$ nvidia-smi -L
```

Append it in `device_ids` list 
```yaml
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              capabilities: [gpu]
              device_ids: ['0'] 
```
Customize Docker Compose YAML file

Change the `docker-compose.yml` file in `vllm` section

```yaml
    command:
      --model ${MODEL_NAME}
      --tensor-parallel-size 2 # Based on GPU count, should be even number of GPUs
      --load-format pt # needed since both `pt` and `safetensors` are available
```




