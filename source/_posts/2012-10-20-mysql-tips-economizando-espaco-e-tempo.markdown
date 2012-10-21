---
layout: post
title: "MySQL Tips: Economizando espaço e tempo"
date: 2012-10-20 14:42
comments: true
categories: [linux,mysql,tips,gzip,sysadmin]
---

Quando faltar espaço em HD ou a transferência de dados de um ponto ao outro não ser 100/100mbit, voce pode realizar a migração do seu banco de dados da seguinte maneira:

<pre>
deadcow@jazz ~ $ mysqldump -u root database | gzip > database.sql.gz
</pre>
e para subir o banco após voce ter transferido para a nova maquina:
<pre>
deadcow@jazz ~ $ gunzip < database.sql.gz | mysql -u root database
</pre>
isto além de economizar o espaço em disco, aumenta a velocidade na transferência do arquivo já que ele vai ser pelo menos 40% menor que o tamanho original.
