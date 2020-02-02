---
layout: post
title: Vault with docker
categories: [cloud]
tags: [postgres, cloud, docker, vault, hashicorp]
fullview: true
comments: false
---

# 1. Run Vault docker
```
docker run --cap-add=IPC_LOCK\
	-e 'VAULT_DEV_ROOT_TOKEN_ID=myroot'\
	-e 'VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200'\
	-e 'VAULT_ADDR=http://127.0.0.1:8200'\
	-e 'VAULT_TOKEN=myroot'\
	-p 8200:8200\
	--name vaul-container\
    vault
```
Testing at http://localhost:8200/ui

# 2. Run Postgres docker
```
docker run -d --name postgres\
    -p 5432:5432\
    -e POSTGRES_PASSWORD=postgres123456\
    -e POSTGRES_USER=postgres \
    postgres
```

# 2. Integration With a Postgres Database
![SignIn](/assets/media/signin-vault-ui.png)

