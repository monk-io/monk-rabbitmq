# Rabbitmq & Monk

This repository contains Monk.io template to deploy sonarqube-community system either locally or on cloud of your choice (AWS, GCP, Azure, Digital Ocean).

This template includes Nginx as a reverse proxy  with rabbitmq out of box.

## Start

Set up Monk - https://docs.monk.io/docs/monk-in-10/

Start `monkd` and login.

```bash
monk login --email=<email> --password=<password>
```

## Clone Monk rabbitmq repository

In order to load templates and change configuration simply use below commands: 
```bash
git clone https://github.com/kaganmersin/monk-rabbitmq

# and change directory to the monk-rabbitmq/rabbitmq template folder
cd monk-rabbitmq/rabbitmq

```

## Configuration

You can add/remove configuration of the template.

The current variables can be found in `rabbitmq/rabbitmq/variables` section

```yaml
  variables:
    rabbitmq-image-tag: "3.10-management"
    nginx-listen-port: 80
    nginx-image-tag: "latest"
    rabbitmq-server-name: "rabbitmq.example.com"
    rabbitmq-node-port: 15672
```

### Rabbitmq configuration files

You can find configuration files in `/files` directory in repository and can edit before the running kit. There are 3 configuration files which bind to the container while run rabbitmq-monk kit 


| Configuration File	 | Format Used | Directory in Container | Purpose 
|----------|-------------|------|---------|
| **rabbitmq.conf** | New style format (sysctl or ini-like)	 | ` /etc/rabbitmq/rabbitmq.conf` | Primary configuration file. Should be used for most settings. It is easier for humans to read and machines (deployment tools) to generate. Not every setting can be expressed in this format.
| **advanced.config** | Classic (Erlang terms) | `/etc/rabbitmq/advanced.config` | A limited number of settings that cannot be expressed in the new style configuration format, such as LDAP queries. Only should be used when necessary. | 
| **rabbitmq-env.conf** | Environment variable pairs | `/etc/rabbitmq/rabbitmq-env.conf` | Used to set environment variables relevant to RabbitMQ in one place. |





##  Template variables

| Variable | Description | Type | Example |
|----------|-------------|------|---------|
| **rabbitmq-node-port** | Rabbitmq user interface port (It must be same with management.tcp.port in rabbitmq.conf  ) | int | 15672
| **rabbitmq-server-name** | Fqdn that nginx will accept and route to. | string | rabbitmq.example.com |
| **rabbitmq-image-tag** | Rabbitmq image version. | string | 3.10-management |
| **nginx-listen-port** | Configures the ports that the nginx listens on. | int | 80 |
| **nginx-image-tag** | Nginx image version. | string | latest |


## Local Deployment

First clone the repository and change the current directory to the /rabbitmq folder and simply run below command after launching `monkd`:
:

```bash
➜  monk load MANIFEST

✨ Loaded:
 ├─🔩 Runnables:
 │  ├─🧩 rabbitmq/nginx
 │  └─🧩 rabbitmq/rabbitmq
 ├─🔗 Process groups:
 │  └─🧩 rabbitmq/stack
 └─⚙️ Entity instances:
    └─🧩 rabbitmq/rabbitmq/metadata
✔ All templates loaded successfully

➜  monk list rabbitmq

✔ Got the list
Type      Template                         Repository  Version      Tags
runnable  nginx/latest                     rabbitmq    -            -
runnable  nginx/reverse-proxy              rabbitmq    -            -
runnable  nginx/reverse-proxy-ssl-certbot  rabbitmq    1.15-alpine  -
runnable  rabbitmq/nginx                   local       -            -
runnable  rabbitmq/rabbitmq                local       -            -
group     rabbitmq/stack                   local       -            -

➜  monk run rabbitmq/stack

✔ Started local/rabbitmq/stack

```

This will start the entire rabbitmq/stack with a Nginx reverse proxy. 


## Cloud Deployment

To deploy the above system to your cloud provider, create a new Monk cluster and provision your instances.

```bash
➜  monk cluster new
? New cluster name rabbitmq
✔ Cluster created
Your cluster has been created successfully.

➜  monk cluster provider add -p gcp -f <path/to/your-key.json>
✔ Provider added successfully

➜  monk cluster grow -p gcp
? Cloud provider gcp
? Name of the new instance my-instance
? Tags (split by whitespace) rabbitmq
? Region europe-central2
? Zone europe-central2-a
? Instance type e2-medium
? Number of instances (or press ENTER to use default = 1) 3
? Default disk type for gcp is HDD (pd-standard). Would you like to change it? No
? Disk size (or press ENTER to use default = 250 GBs) 50
✔ Start creation of new instance(s) on gcp... DONE
✔ Creating node: my-instance-1 DONE
✔ Initializing node: my-instance-1 DONE
✔ Creating node: my-instance-2 DONE
✔ Initializing node: my-instance-2 DONE
✔ Creating node: my-instance-3 DONE
✔ Initializing node: my-instance-3 DONE
✔ Connecting: my-instance-1 DONE
✔ Syncing peer: my-instance-1 DONE
✔ Connecting: my-instance-2 DONE
✔ Connecting: my-instance-3 DONE
✔ Syncing peer: my-instance-2 DONE
✔ Syncing peer: my-instance-3 DONE
✔ Syncing nodes DONE
✔ Cluster grown successfully
```

Once cluster is ready execute the same command as for local and select your cluster (the option will appear automatically).
```bash
➜  monk load MANIFEST

✨ Loaded:
 ├─🔩 Runnables:
 │  ├─🧩 rabbitmq/nginx
 │  └─🧩 rabbitmq/rabbitmq
 ├─🔗 Process groups:
 │  └─🧩 rabbitmq/stack
 └─⚙️ Entity instances:
    └─🧩 rabbitmq/rabbitmq/metadata
✔ All templates loaded successfully

➜  monk list rabbitmq

✔ Got the list
Type      Template                         Repository  Version      Tags
runnable  nginx/latest                     rabbitmq    -            -
runnable  nginx/reverse-proxy              rabbitmq    -            -
runnable  nginx/reverse-proxy-ssl-certbot  rabbitmq    1.15-alpine  -
runnable  rabbitmq/nginx                   local       -            -
runnable  rabbitmq/rabbitmq                local       -            -
group     rabbitmq/stack                   local       -            -

➜  monk run rabbitmq/stack

✔ Started local/rabbitmq/stack

```

## Logs & Shell

```bash
# show Rabbitmq logs
➜  monk logs -l 1000 -f rabbitmq/rabbitmq

# show Nginx logs
➜  monk logs -l 1000 -f rabbitmq/nginx

# access shell in the container running Rabbitmq
➜  monk shell rabbitmq/rabbitmq

# access shell in the container running Nginx
➜  monk shell rabbitmq/nginx

```

## Stop, remove and clean up workloads and templates

```bash
➜ monk purge -x rabbitmq/stack rabbitmq/rabbitmq rabbitmq/nginx 

✔ rabbitmq/stack purged
✔ rabbitmq/rabbitmq purged
✔ rabbitmq/nginx purged

```