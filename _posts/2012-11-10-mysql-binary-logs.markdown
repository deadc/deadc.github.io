---
layout: post
title: mysql binary logs
subtitle: porque meu HD está cheio?
date: 2012-11-10 14:34
comments: true
categories: [tips, devops]
---

Se o diretório onde está alocado os arquivos data de seu MySQL está crescendo mais que o normal, possivelmente você está tendo problemas com a configuração quanto a flag que determina o uso dos binlogs.

Binlogs em sua essência são utilizados para realizar a replicação de bancos de dados e em algumas ocasiões te salvar quando algum dado é perdido, se você não realiza a replicação e tampouco se preocupa com o armazenamento dos binlogs, não existe sentido manter essa configuração.

Para desabilitar esta opção, comente a linha no seu `/etc/mysql/my.cnf` que diz `log-bin=mysql-bin` onde define que os binlogs devem ser criados e indica qual nome dos arquivos. Você ainda pode parar a geração dos binlogs _on-the-fly_ utilizando o seguinte comando como usuario __root__

{% highlight bash %}
mysql> SET GLOBAL SQL_LOG_BIN=0;
Query OK, 0 rows affected (0.00 sec)
{% endhighlight %}

Este comando na verdade não desabilita o binlogs e sim, ignora INSERTs, DDLs, UPDATEs, e DELETEs, para que a alteração seja permanente você ainda terá que editar o `/etc/mysql/my.cnf`.

Caso você faça uso dos binlogs, você ainda pode definir um tamanho e qual a frequencia de criação destes arquivos, ajustando de acordo com a necessidade do uso, utilizando as flags:

* __max_binlog_size__: que define qual tamanho maximo do arquivo em kilobytes. quando atingir este valor será criado um novo arquivo sequencial no formato nome-do-arquivo.XXXXXX

* __expire_logs_days__: que define de quanto em quanto dias os binlogs serão removidos, recomenda-se utilizar o periodo minimo de 10 dias, dependendo do atraso entre o master e slave quando existir a replicação de bancos.

* __binlog-ignore-db__: que especifica quais databases podem ser ignoradas pelo binlog.

* __binlog-do-db__: que especifica quais databases devem ser monitoradas e os comandos salvos no binlog.

Feita a configuração, e tendo certeza que os binlogs armazenados não são mais necessários, você poderá excluir todos os binlogs através do comando

{% highlight bash %}
mysql> RESET MASTER;
Query OK, 0 rows affected (0.00 sec)
{% endhighlight %}

ou se o mysqld estiver configurado como `SLAVE`

{% highlight bash %}
mysql> RESET SLAVE;
Query OK, 0 rows affected (0.06 sec)
{% endhighlight %}

Você ainda poderá especificar quais binlogs devem ser removidos pela sequencia da criação dos arquivos

{% highlight bash %}
mysql> PURGE BINARY LOGS TO 'mysql-bin.000015';
Query OK, 0 rows affected (0.08 sec)
{% endhighlight %}

ou através da data de criação

{% highlight bash %}
mysql> PURGE BINARY LOGS BEFORE '20012-11-10 00:00:00';
Query OK, 0 rows affected (0.11 sec)
{% endhighlight %}

Ou mesmo utilizando o `mysqladmin` com o parametro `flush-logs`, que exclui qualquer binlog anterior a 3 dias

{% highlight bash %}
jazz//deadcow ~ % mysqladmin -u root flush-logs
{% endhighlight %}

A remoção dos binlogs devem ser realizadas através do proprio MySQL, remoções diretas podem causar efeitos inesperados no comportamento normal do serviço.

A geração de logs pelo mysqld pode ser util ao identificar problemas de performance, porém devem ser tratados com certa cautela para que uma solução não vire uma dor de cabeça. ainda existem outros fatores que podem contribuir para o aumento do uso do disco pelo mysqld e que serão abordados em outros posts no futuro.
