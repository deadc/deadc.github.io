---
layout: post
title: "ssh-agent vs ssh_config"
subtitle: organizando seus acessos
date: 2012-10-20 18:17
comments: true
categories: [tips, devops]
---

Muitas vezes utilizamos chaves privadas para acessar servidores remotos via ssh, principalmente agora com a utilização de autenticações somente por chave como por exemplo a Engine Yard, que roda em cima do PaaS da amazon (AWS).

A maioria das pessoas costuma utilizar o `ssh-agent` como gerenciador dessas chaves, quase todos os tutoriais passo-a-passo a respeito de acesso remoto via ssh sem a utilização de senhas mencionam ele.

O problema é que nem todas as distribuições oferecem ele como pacote padrão, além disso, o ssh-agent adiciona apenas uma vez a chave a sessão, ele não vai requerer novamente a senha caso exista, isto pode ser visto como um problema de segurança.

Uma alternativa ao ssh-agent que pouca gente conhece é a associação do host destino a uma chave através do ssh_config, cada usuário tem a opção de criar um ssh_config próprio, para determinar o comportamento do cliente ssh.

Por padrão, o cliente ssh irá buscar a chave no caminho `~/.ssh/id_rsa` e `~/.ssh_dsa` para hosts que utilizem a versão 2 do protocolo ssh ou `~/.ssh/identity` para hosts que utilizem o protocolo 1, para alterar este comportamento definindo um arquivo para cada host, edite ou crie o arquivo `~/.ssh/config` adicionando as linhas:

{% highlight bash %}
Host ssh.deadc.org
  IdentityFile ~/.ssh/id_rsa-deadc.org
{% endhighlight %}

sendo que o arquivo `id_rsa-deadc.org` é a nossa chave que utilizaremos para acesso o servidor remoto, além de definir uma chave para cada host, é possivel definir também para um grupo de host:

{% highlight bash %}
Host *.deadc.org
  IdentityFile ~/.ssh/id_rsa-deadc.org
{% endhighlight %}

ou mesmo para todos os hosts:

{% highlight bash %}
Host *
  IdentityFile ~/.ssh/id_rsa-deadc.org
{% endhighlight %}

Você poderá especificar mais de uma chave para cada host, cada chave será enviada para o servidor em sequencia, além disso, caso o ssh-agent esteja rodando e contenha alguma outra chave, ela também será enviada ao host na tentativa de se autenticar com o servidor. leia o `man ssh_config` para mais opções e informações a respeito.
