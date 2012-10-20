---
layout: post
title: "MySQL: Binlogs Backup full e incremental"
date: 2012-10-20 15:11
comments: true
categories: [linux, mysql, mysqlbinlog, mysqldump, sysadmin, backup]
---

Mysql binlogs são logs de todos os comandos enviados para o mysql, ele captura todos os comandos executados, desde um simples INSERT até um DROP DATABASE e insere indices atraves de datas e numeros sequenciais. É uma ferramenta muito poderosa, utilizada na replicação de banco de dados e também na realização de backups incrementais.

Por padrão, diversas distribuições linux deixam essa opção desabilitada nos pacotes tradicionais do mysql-server, se este for o caso da sua instalação, habilite o binlog no seu servidor

{% codeblock /etc/mysql/my\.cnf %}
log_bin                 = /var/lib/mysql/mysql-bin.log
expire_logs_days        = 10
{% endcodeblock %}

O ``expire_logs_days`` é a quantidade de dias que os bin-logs devem permanecer no servidor antes que sejam deletados, já na primeira linha, indica onde os bin-logs serão salvos, neste caso em ``/var/lib/mysql``, é uma boa idéia separar a pasta onde os arquivos serão salvos caso você deseje realizar o backup incremental.
<pre>
[root@jazz ~]# ls -la /var/lib/mysql/mysql-bin.*
-rw-rw---- 1 mysql mysql 27287 Jul 15 15:02 /var/lib/mysql/mysql-bin.000001
-rw-rw---- 1 mysql mysql 35873 Jul 15 15:02 /var/lib/mysql/mysql-bin.000002
-rw-rw---- 1 mysql mysql   107 Jul 15 15:03 /var/lib/mysql/mysql-bin.000003
-rw-rw---- 1 mysql mysql    57 Jul 15 15:03 /var/lib/mysql/mysql-bin.index
[root@jazz ~]#
</pre>
Embora no diretorio destino exista mais de um binlog, apenas um está ativo no momento, você pode verificar qual e em qual posição o mesmo se encontra executando o seguinte comando:
<pre>
deadcow@jazz ~ $ mysql -u root -e 'SHOW MASTER STATUS;'
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000003 |      107 |              |                  |
+------------------+----------+--------------+------------------+
deadcow@jazz ~ $
</pre>
Agora iremos criar uma database para exemplificar o uso do binlog
{% codeblock lang:sql %}
CREATE DATABASE `bk_test`;
USE `bk_test`;
CREATE TABLE `bk_test_t1` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `test_field` VARCHAR(30) NOT NULL,
    `time_created` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB;
{% endcodeblock %}
e então inserir algumas entradas
{% codeblock lang:sql %}
USE `bk_test`;
INSERT into `bk_test_t1` (test_field) VALUES ('val1');
INSERT into `bk_test_t1` (test_field) VALUES ('val2');
INSERT into `bk_test_t1` (test_field) VALUES ('val3');
{% endcodeblock %}
Agora temos um banco de dados ativo com entradas validas para a realização de um backup, primeiro iremos realizar um full backup em nosso banco
<pre>
mysqldump -uroot -p --all-databases --single-transaction --flush-logs \
--master-data=2 > full_backup.sql
</pre>
A flag ``–flush-logs`` é utilizada para que o binlog ativo seja fechado e um novo seja criado, ``–master-data=2`` adiciona comentários no dump com os indices do binlog, já a flag ``–single-transaction`` é utilizada para a realização consistente de dumps de bancos InnoDB, para quem não está familiarizado, uma forma simples de explicar uma transaction é dizer que é um unico comando, caso alguma entrada falhe, o comando inteiro será descartado e nenhuma alteração será feita no banco.
<pre>
[root@jazz ~]# ls -la /var/lib/mysql/mysql-bin.*
-rw-rw---- 1 mysql mysql 27287 Jul 15 15:02 /var/lib/mysql/mysql-bin.000001
-rw-rw---- 1 mysql mysql 35873 Jul 15 15:02 /var/lib/mysql/mysql-bin.000002
-rw-rw---- 1 mysql mysql  1208 Jul 15 15:13 /var/lib/mysql/mysql-bin.000003
-rw-rw---- 1 mysql mysql    57 Jul 15 15:03 /var/lib/mysql/mysql-bin.index
[root@jazz ~]# mysqldump -uroot -p --all-databases --single-transaction \
> --flush-logs --master-data=2 > full_backup.sql
Enter password:
[root@jazz ~]# ls -la /var/lib/mysql/mysql-bin.*
-rw-rw---- 1 mysql mysql 27287 Jul 15 15:02 /var/lib/mysql/mysql-bin.000001
-rw-rw---- 1 mysql mysql 35873 Jul 15 15:02 /var/lib/mysql/mysql-bin.000002
-rw-rw---- 1 mysql mysql  1251 Jul 15 15:26 /var/lib/mysql/mysql-bin.000003
-rw-rw---- 1 mysql mysql   107 Jul 15 15:26 /var/lib/mysql/mysql-bin.000004
-rw-rw---- 1 mysql mysql    76 Jul 15 15:26 /var/lib/mysql/mysql-bin.index
[root@jazz ~]#
</pre>
Agora iremos adicionar algumas entradas novas ao banco para exemplificar o backup incremental
{% codeblock lang:sql %}
USE `bk_test`;
INSERT into `bk_test_t1` (test_field) VALUES ('val4');
INSERT into `bk_test_t1` (test_field) VALUES ('val5');
INSERT into `bk_test_t1` (test_field) VALUES ('val6');
{% endcodeblock %}
Como realizamos o flush-logs no backup full, as novas entradas devem estar no arquivo mysql-bin.000004, para realizarmos o backup incremental de forma consistente, precisamos iniciar um novo binlog e copiar o arquivo (ou arquivos) para outra localidade, para iniciar um novo binlog execute
<pre>
[root@jazz ~]# mysqladmin -uroot -p flush-logs
[root@jazz ~]# ls -la /var/lib/mysql/mysql-bin.*
-rw-rw---- 1 mysql mysql 27287 Jul 15 15:02 /var/lib/mysql/mysql-bin.000001
-rw-rw---- 1 mysql mysql 35873 Jul 15 15:02 /var/lib/mysql/mysql-bin.000002
-rw-rw---- 1 mysql mysql  1251 Jul 15 15:26 /var/lib/mysql/mysql-bin.000003
-rw-rw---- 1 mysql mysql   885 Jul 15 15:38 /var/lib/mysql/mysql-bin.000004
-rw-rw---- 1 mysql mysql   107 Jul 15 15:38 /var/lib/mysql/mysql-bin.000005
-rw-rw---- 1 mysql mysql    95 Jul 15 15:38 /var/lib/mysql/mysql-bin.index
[root@jazz ~]#
</pre>
Agora temos os componentes para realizar um restore a partir de um backup full e um incremental, nosso banco agora está desta forma
<pre>
mysql> USER bk_test;
Database changed

mysql> SHOW TABLES;
+-------------------+
| Tables_in_bk_test |
+-------------------+
| bk_test_t1        |
+-------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM bk_test_t1;
+----+------------+---------------------+
| id | test_field | time_created        |
+----+------------+---------------------+
| 1  | val1       | 2012-07-15 15:13:53 |
| 2  | val2       | 2012-07-15 15:13:53 |
| 3  | val3       | 2012-07-15 15:13:54 |
| 4  | val4       | 2012-07-15 15:33:50 |
| 5  | val5       | 2012-07-15 15:33:50 |
| 6  | val6       | 2012-07-15 15:33:52 |
+----+------------+---------------------+
6 rows in set (0.00 sec)

mysql>
</pre>
Digamos agora que acidentalmente a database que utilizamos foi completamente removida
<pre>
mysql> DROP DATABASE bk_test;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

mysql>
</pre>
Tudo bem! temos o backup full e estamos a salvo certo?!
<pre>
deadcow@jazz ~ $ mysql -u root < full_backup.sql
deadcow@jazz ~ $ mysql -u root
mysql> use bk_test;
Database changed

mysql> SELECT * FROM bk_test_t1;
+----+------------+---------------------+
| id | test_field | time_created        |
+----+------------+---------------------+
| 1  | val1       | 2012-07-15 15:13:53 |
| 2  | val2       | 2012-07-15 15:13:53 |
| 3  | val3       | 2012-07-15 15:13:54 |
+----+------------+---------------------+
3 rows in set (0.00 sec)

mysql>
</pre>
Este backup não está completo, havia 6 entradas e agora só existem 3, mas ainda temos o backup incremental de um periodo posterior ao backup full, vamos aplica-lo
<pre>
[root@jazz ~]# cd /var/lib/mysql/
[root@jazz mysql]# mysqlbinlog mysql-bin.000004 | mysql -u root bk_test
[root@jazz mysql]# mysql -u root bk_test

mysql> SELECT * FROM bk_test_t1;
+----+------------+---------------------+
| id | test_field | time_created        |
+----+------------+---------------------+
| 1  | val1       | 2012-07-15 15:13:53 |
| 2  | val2       | 2012-07-15 15:13:53 |
| 3  | val3       | 2012-07-15 15:13:54 |
| 4  | val4       | 2012-07-15 15:33:50 |
| 5  | val5       | 2012-07-15 15:33:50 |
| 6  | val6       | 2012-07-15 15:33:52 |
+----+------------+---------------------+
6 rows in set (0.00 sec)

mysql>
</pre>
E tudo está recuperado, sem dores de cabeça. a ferramenta ``mysqlbinlog`` acompanha o pacote de instalação do ``mysql-server``, ela retorna os comandos do binlog em plaintext e através dela é possivel recuperar o banco de dados, também pode se redirecionar a saida do programa para um arquivo e então editá-lo para excluir queries destrutivas antes de executar no mysql.

Além disso é possivel passar como parametro um periodo especifico de tempo para o mysqlbinlog, é muito util para recuperar pequenas quantidades de entradas perdidas em um banco de dados, desde que se conheca o periodo.
<pre>
mysqlbinlog --start-datetime="2012-07-15 15:13:53" \ 
--stop-datetime="2012-07-15 15:13:54" mysql-bin.000004|mysql -uroot bk_test
</pre>
Existem outras dezenas de possibilidades no MySQL que poucas pessoas conhecem, procure blogs, assine feeds e procure crescer profissionalmente através do seu conhecimento, facilite sua vida e a da sua empresa!

Referencias:

* [Planet MySQL](http://planet.mysql.com)
* [Mysql Backup and point in time recovery with-binary logs](http://www.bytetouch.com/blog/system-administration/mysql-backup-and-point-in-time-recovery-with-binary-logs)
* [Backing up binary log files with mysqlbinlog](http://www.mysqlperformanceblog.com/2012/01/18/backing-up-binary-log-files-with-mysqlbinlog)

