---
layout: post
title: "Instalacao Ansible Tower em Debian 9 Stretch"
date: 2017-09-08 05:46
comments: true
categories: ansible, iac, tower, awx
---

Recentemente a Red Hat disponibilizou de forma Open Source o codigo fonte do Ansible Tower, ferramenta web que gerencia as execucoes de playbooks do ansible através do seu novo projeto AWX.

Este é um passo-a-passo de como instalar o Ansible Tower no Debian 9 Stretch rodando em container Docker, e segue basicamente a documentacao oficial do projeto porém em uma instalacao para o Debian 9 Stretch.
A instalacao das dependencias é simples, o Ansible Tower tem sua UI em nodejs, a instalacao do gettext é uma dependencia para o comando msgfmt, necessário durante a build.

	# curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
	# apt-get install nodejs gettext docker
	# pip install ansible docker-py # (ansible 2.3)

Apos a instalacao das dependencias, o processo de build dos containers é totalmente automatizado pelo proprio Ansible

	$ git clone git@github.com:ansible/awx.git
	# cd awx/installer && ansible-playbook -i inventory install.yml

Aguarde o termino do build e entao, no comando `docker ps` os containers em execucao deveram parecer como estes

	CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
	e240ed8209cd awx_task:1.0.0.8 “/tini — /bin/sh …” 2 minutes ago Up About a minute 8052/tcp awx_task
	1cfd02601690 awx_web:1.0.0.8 “/tini — /bin/sh …” 2 minutes ago Up About a minute 0.0.0.0:80->8052/tcp awx_web
	55a552142bcd memcached:alpine “docker-entrypoint…” 2 minutes ago Up 2 minutes 11211/tcp memcached
	84011c072aad rabbitmq:3 “docker-entrypoint…” 2 minutes ago Up 2 minutes 4369/tcp, 5671–5672/tcp, 25672/tcp rabbitmq
	97e196120ab3 postgres:9.6 “docker-entrypoint…” 2 minutes ago Up 2 minutes 5432/tcp postgres

Após isso, confira a execucao das migrations com o comando `docker logs -f awx_task`

	Using /etc/ansible/ansible.cfg as config file
	127.0.0.1 | SUCCESS => {
	 “changed”: false,
	 “db”: “awx”
	}
	Operations to perform:
	 Synchronize unmigrated apps: solo, api, staticfiles, messages, channels, django_extensions, ui, rest_framework, polymorphic
	 Apply all migrations: sso, taggit, sessions, djcelery, sites, kombu_transport_django, social_auth, contenttypes, auth, conf, main
	Synchronizing apps without migrations:
	 Creating tables…
	 Running deferred SQL…
	 Installing custom SQL…
	Running migrations:
	 Rendering model states… DONE
	 Applying contenttypes.0001_initial… OK
	 Applying contenttypes.0002_remove_content_type_name… OK
	 Applying auth.0001_initial… OK
	 Applying auth.0002_alter_permission_name_max_length… OK
	 Applying auth.0003_alter_user_email_max_length… OK
	 Applying auth.0004_alter_user_username_opts… OK
	 Applying auth.0005_alter_user_last_login_null… OK
	 Applying auth.0006_require_contenttypes_0002… OK
	 Applying taggit.0001_initial… OK
	 Applying taggit.0002_auto_20150616_2121… OK
	 Applying main.0001_initial… OK

Voce deverá ver algo parecido com esta mensagem, indicando que o processo de instalacao terminou

	Python 2.7.5 (default, Nov 6 2016, 00:28:07)
	[GCC 4.8.5 20150623 (Red Hat 4.8.5–11)] on linux2
	Type “help”, “copyright”, “credits” or “license” for more information.
	(InteractiveConsole)
	>>> <User: admin>
	>>> Default organization added.
	Demo Credential, Inventory, and Job Template added.
	Successfully registered instance awx
	(changed: True)
	Creating instance group tower
	Added instance awx to tower
	(changed: True)

O acesso ao painel web deverá ser realizado no endereco http://localhost, com usuario **admin** e senha **password**.

