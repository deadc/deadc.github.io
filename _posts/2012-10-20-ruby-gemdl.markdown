---
layout: post
title: "ruby: gem dl"
subtitle: "acessando funções externas"
date: 2012-10-20 14:49
comments: true
categories: [dev, devops]
---

Eu estava precisando acessar uma DLL especifica no Windows para automatizar um processo no serviço, eu (ainda!) não domino muito a linguagem ruby, mas por conta de todos meus colegas de trabalho fazerem uso, inclusive do RoR meio que fui absorvido por essa linguagem que, depois de python virou uma das minhas preferidas.

Pois bem, depois de algumas tentativas frustradas na sexta-feira, resolvi no sábado pesquisar melhor como em ruby, acessar funções externas de dll’s e lib’s. meu maior problema na sexta era a falta de compatibilidade de versões do ruby, eu escrevia um código pra ruby 1.9.2 e tentava rodar em uma versão anterior, 1.8.7 (Windows em produção).

Em casa, consegui escrever 2 scripts para versões distintas e finalmente criar o script para acessar a DLL, o script é tão simples que cheguei seriamente a pensar em não publicar isso, mas como pode ser util para alguém, segue ai!

{% highlight ruby %}
require ‘dl’

dl = DL.dlopen(‘libcurl.so’)
mycall = dl["curl_version",'S']
puts mycall.call
{% endhighlight %}

E a versão para Ruby 1.9.2, que é um pouquinho diferente, mas tão facil quanto:

{% highlight ruby %}
require ‘dl’
require ‘fiddle’

dl = DL.dlopen(‘libcurl.so’)
mycall = Fiddle::Function.new(dl['curl_version'], [], Fiddle::TYPE_VOIDP)
puts mycall.call
{% endhighlight %}

Lembrando que por conta do uso do Fiddle, este exemplo só funciona no Ruby 1.9.3 ou superior, para ser 100% compativel com 1.9 você deve fazer as modificações necessárias.

Explicando: no primeiro script onde se lê `curl_version`` é aonde vai o nome da função externa, que a lib ou dll contém, no segundo campo, é o tipo de entrada da função mais o tipo de retorno, se a função recebesse algum argumento como por exemplo do tipo String e retornasse um Inteiro, ficaria assim:

{% highlight ruby %}
mycall = dl["funcao",'SI']
puts mycall.call(“string”)
{% endhighlight %}

no segundo script isso fica mais claro já que, o retorno fica no final, enquanto os argumentos dentro de um array, que neste caso está vazio já que a função não necessita de nenhum parametro. Mais referencias de como usar e sobre os tipos possiveis de entrada nas funções RTFM

uma dica para quem usa linux e quer saber quais funções determinada lib possui e não tem o código-fonte, é utilizar o comando nm:

{% highlight bash %}
deadcow@stack ~ $ nm –defined-only -D /usr/lib/libcurl.so|grep easy
00000000000297d0 T Curl_easy_addmulti
0000000000029b40 T Curl_easy_initHandleData
00000000000319c0 T Curl_multi_rmeasy
0000000000040270 T Curl_pp_easy_statemach
0000000000029e30 T curl_easy_cleanup
0000000000029c00 T curl_easy_duphandle
0000000000022de0 T curl_easy_escape
0000000000029e00 T curl_easy_getinfo
000000000002a0d0 T curl_easy_init
00000000000299b0 T curl_easy_pause
0000000000029e50 T curl_easy_perform
0000000000029930 T curl_easy_recv
0000000000029b60 T curl_easy_reset
0000000000029880 T curl_easy_send
0000000000029f40 T curl_easy_setopt
0000000000034e30 T curl_easy_strerror
0000000000022cb0 T curl_easy_unescape
deadcow@stack ~ $
{% endhighlight %}

obviamente consultando o oráculo depois para saber mais sobre a função!

