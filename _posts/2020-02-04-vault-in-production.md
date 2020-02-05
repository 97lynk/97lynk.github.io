---
layout: post
title: Quick guide to Vault in production
categories: [cloud, spring]
tags: [cloud, docker, vault, hashicorp, spring]
description: Deploying Vault in production environment.
comments: false
---

# 1. Run Vault i production mode
Let's overide Vault using `VAULT_LOCAL_CONFIG` variable with json format
{% highlight shell %}
$ docker run --cap-add=IPC_LOCK \
	-p 8200:8200 \
	-v ./vault:/vault/file \
	-e 'VAULT_ADDR=http://127.0.0.1:8200' \
	-e 'VAULT_LOCAL_CONFIG={"api_addr": "http://127.0.0.1:8200", "listener": [{ "tcp": { "address": "0.0.0.0:8200", "tls_disable": 1 } } ], "storage": { "file": { "path": "/vault/file" } }, "max_lease_ttl": "10h", "default_lease_ttl": "10h", "cluster_name":"testcluster", "ui":true }' \
	--name vault-server-mode \
	vault server
{% endhighlight %}

See more details in <a href="https://www.vaultproject.io/docs/configuration/" target="_blank">Vault's documents</a>

Note that:
* We can configure where store sensitive infomations
* Root token hadn't generated when start vault
* We must unseal vault to use it

Testing at <a href="http://localhost:8200/ui" target="_blank">http://localhost:8200/ui</a>

# 2. Unsealing Vault
## 2.1. Access to Vault container
{% highlight shell %}
$ docker exec -it vault-server-mode sh
{% endhighlight %}

## 2.2. Initializing Vault
{% highlight shell %}
$ vault operator init
{% endhighlight %}

Result look like

![Initializing Vault](/assets/media/init-vault.png)

## 2.3. Unsealing
Unseal by 3 unseal key in above
{% highlight shell %}
$ vault operator unseal
{% endhighlight %}