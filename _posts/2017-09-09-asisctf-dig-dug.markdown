---
layout: post
title: "asisctf: dig dug"
subtitle: "ASIS CTF Finals 2017" 
date: 2017-09-08 05:46
comments: true
categories: [ctfs]
---

---

O desafio apresentava um endpoint digx.asisctf.com na categoria web, e dava algumas dicas que o a resposta poderia estar no comand dig, ou seja no dominio de onde a challenge era apresentado

{% highlight bash %}
deadcow@capthook:~$ dig digx.asisctf.com any +multiline +noall +answer

; <<>> DiG 9.10.3-P4-Debian <<>> digx.asisctf.com any +multiline +noall +answer
;; global options: +cmd
digx.asisctf.com.       300 IN A 192.81.223.250
digx.asisctf.com.       300 IN RRSIG A 7 3 300 (
                                20171007125525 20170907125525 182 asisctf.com.
                                aoO35AqtS3IBePvCKiH2FRauTzj2HZ8GgNdS44Z35vUm
                                qAxTfj2spP7s+JBGU3dytDSJ2PJZe4MqYTTOkHwEWu+M
                                /xvhCv4kVk5dM2uhvp3evVmQFhnMfLCPhu7XP6A39LVB
                                Zi3SfAKMqv7/GyoT9+qnUyMBLiIHmObwOlptEeSrBRLV
                                qT8vm7AIY7ZpuoreiY3vjbywrGIjrAqYG5puoppqSWGT
                                i9UD3pTEdnAZZZnElc0vJ7c+r6ENHmtF7riXvorR8RyB
                                ASkoXpI0uqsaE17IyYUKj1cHlstHB7FNtsygekk1fwtn
                                IM0u8qNHsWbZUEtODuR/vBkKPjMdLRlIfQ== )
deadcow@capthook:~$
{% endhighlight %}

O dominio da challenge, era digx.asisctf.com, e dig -x resolve apenas por IP, na verdade ele me indica quando feito no dominio, os endpoints do root-servers, porem resolvi tentar para ver se achava alguma pista

{% highlight bash %}
deadcow@capthook:~$ dig -x 192.81.223.250

; <<>> DiG 9.10.3-P4-Debian <<>> -x 192.81.223.250
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1456
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;250.223.81.192.in-addr.arpa.   IN      PTR

;; ANSWER SECTION:
250.223.81.192.in-addr.arpa. 877 IN     PTR     airplane.asisctf.com.

;; Query time: 2 msec
;; SERVER: 67.207.67.3#53(67.207.67.3)
;; WHEN: Fri Sep 08 19:17:23 UTC 2017
;; MSG SIZE  rcvd: 90

deadcow@capthook:~$
{% endhighlight %}

acessando o endpoint http://airplane.asisctf.com ele me pedia para entrar em modo offline para ver a flag, acessando pelo celular em modo airplane, após um breve texto, a flag: **ASIS{_just_Go_Offline_When_you_want_to_be_creative_!}**


