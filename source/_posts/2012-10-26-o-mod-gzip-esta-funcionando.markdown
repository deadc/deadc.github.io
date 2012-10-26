---
layout: post
title: "O mod_gzip está funcionando?"
date: 2012-10-26 17:58
comments: true
categories: [linux,wget,sysadmin,gzip]
---

Um jeito simples de verificar se seu site está realmente enviando paginas comprimidas para o cliente, sem ter que instalar um plugin no seu browser é utilizar o parametro ``--header`` do wget.

{% blockquote man wget, retirado do manual pages %}
--header=header-line
	Send header-line along with the rest of the headers in each HTTP request.  The supplied header is sent as-is, which means it must contain name and value separated by colon, and must not contain newlines.
{% endblockquote %}

Passando ao parametro ``--header`` do wget ``Accept-Encoding: gzip`` é possivel baixar a página comprimida, claro se o servidor tiver esta opção

<pre>
deadcow@jazz ~ $ wget --header='Accept-Encoding: gzip' \
http://www.uol.com.br -O sample.gz
--2012-10-26 18:07:56--  http://www.uol.com.br/
Resolving www.uol.com.br... 200.221.2.45, 200.147.67.142
Connecting to www.uol.com.br|200.221.2.45|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘sample.gz’

    [  <=>                               ] 86,339       211KB/s   in 0.4s

2012-10-26 18:07:57 (211 KB/s) - ‘sample.gz’ saved [86339]

deadcow@jazz ~ $ file sample.gz
sample.gz: gzip compressed data, from Unix
deadcow@jazz ~ $
</pre>
Já se o servidor não oferece este recurso, o comando ``file`` retornará uma página html normal
<pre>
deadcow@jazz ~ $ wget --header='Accept-Encoding: gzip' \
http://www.blunt.com.br -O sample.gz
--2012-10-26 18:08:17--  http://www.blunt.com.br/
Resolving www.blunt.com.br... 216.14.115.179
Connecting to www.blunt.com.br|216.14.115.179|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘sample.gz’

    [     <=>                            ] 45,874      53.6KB/s   in 0.8s

2012-10-26 18:08:19 (53.6 KB/s) - ‘sample.gz’ saved [45874]

deadcow@jazz ~ $ file sample.gz
sample.gz: HTML document, ISO-8859 text, with very long lines, \
with CR, LF line terminators
deadcow@jazz ~ $
</pre>

Além do ``Accept-Encoding: gzip``, você pode passar qualquer parametro para o servidor, substituindo os valores padrões do wget, muito util para testar alguns redirecionamentos

<pre>
deadcow@jazz ~ $ wget --header='Host: ig.com.br' \
http://www.uol.com.br
--2012-10-26 18:14:45--  http://www.uol.com.br/
Resolving www.uol.com.br... 200.221.2.45, 200.147.67.142
Connecting to www.uol.com.br|200.221.2.45|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.uol.com.br/ [following]
[...]
Connecting to www.uol.com.br|200.221.2.45|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.uol.com.br/ [following]
--2012-10-26 18:14:47--  http://www.uol.com.br/
Connecting to www.uol.com.br|200.221.2.45|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.uol.com.br/ [following]
20 redirections exceeded.
deadcow@jazz ~ $
</pre>

Dica retirada do site [nixcraft](http://www.cyberciti.biz/faq/unix-linux-wget-download-compressed-gzip-headers)

