---
layout: post
title: "gzip & mysqldump"
subtitle: economizando o espaco e o tempo
date: 2012-10-20 14:42
comments: true
categories: [tips, devops]
---

Quando faltar espaço em HD ou a transferência de dados de um ponto ao outro não ser 100/100mbit, voce pode realizar a migração do seu banco de dados da seguinte maneira:

{% highlight bash %}
deadcow@jazz ~ $ mysqldump -u root database | gzip > database.sql.gz
{% endhighlight %}

e para subir o banco após voce ter transferido para a nova maquina:

{% highlight bash %}
deadcow@jazz ~ $ gunzip < database.sql.gz | mysql -u root database
{% endhighlight %}

isto além de economizar o espaço em disco, aumenta a velocidade na transferência do arquivo já que ele vai ser pelo menos 40% menor que o tamanho original.
