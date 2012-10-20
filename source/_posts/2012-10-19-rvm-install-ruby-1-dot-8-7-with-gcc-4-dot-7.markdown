---
layout: post
title: "RVM install REE/Ruby 1.8.7 with GCC 4.7"
date: 2012-10-19 03:09
comments: true
categories: [gcc,bundle,linux,ruby,rvm]
---

Na instalação do ree/ruby 1.8.7 pelo RVM nas versões mais novas do gcc (estou utilizando a versão 4.7.2), o seguinte erro é apresentado quando você tenta rodar ``gem install bundler`` (não somente este comando, mas alguns outros também):
```
deadcow@jazz ~ $ gem install bundler
/home/deadcow/.rvm/rubies/ree-1.8.7-2012.02/lib/ruby/1.8/timeout.rb:60: [BUG] Segmentation fault
ruby 1.8.7 (2012-02-08 MBARI 8/0x6770 on patchlevel 358) [x86_64-linux], MBARI 0x6770, Ruby Enterprise Edition 2012.02
```
para resolver é muito simples, faça o seguinte procedimento:
```
deadcow@jazz ~ $ rvm remove ree
Removing /home/deadcow/.rvm/src/ree-1.8.7-2012.02...
Removing /home/deadcow/.rvm/rubies/ree-1.8.7-2012.02...
Removing ree-1.8.7-2012.02 aliases...
Removing ree-1.8.7-2012.02 wrappers...
Removing ree-1.8.7-2012.02 environments...
Removing ree-1.8.7-2012.02 binaries...
deadcow@jazz ~ $ export CFLAGS="-O2 -fno-tree-dce -fno-optimize-sibling-calls"
deadcow@jazz ~ $ rvm install ree
Installing Ruby Enterprise Edition from source to: /home/deadcow/.rvm/rubies/ree-1.8.7-2012.02
ree-1.8.7-2012.02 - #fetching (ruby-enterprise-1.8.7-2012.02)
ree-1.8.7-2012.02 - #extracting ruby-enterprise-1.8.7-2012.02 to /home/deadcow/.rvm/src/ree-1.8.7-2012.02
Applying patch 'tcmalloc' (located at /home/deadcow/.rvm/patches/ree/1.8.7/tcmalloc.patch)
Applying patch 'stdout-rouge-fix' (located at /home/deadcow/.rvm/patches/ree/1.8.7/stdout-rouge-fix.patch)
Applying patch 'no_sslv2' (located at /home/deadcow/.rvm/patches/ree/1.8.7/no_sslv2.diff)
Applying patch 'lib64' (located at /home/deadcow/.rvm/patches/ree/lib64.patch)
ree-1.8.7-2012.02 - #installing
Removing old Rubygems files...
Installing rubygems-1.8.24 for ree-1.8.7-2012.02 ...
Installation of rubygems completed successfully.
ree-1.8.7-2012.02 - #importing default gemsets (/home/deadcow/.rvm/gemsets/)
deadcow@jazz ~ $ gem install bundler
Fetching: bundler-1.2.1.gem (100%)
Successfully installed bundler-1.2.1
1 gem installed
Installing ri documentation for bundler-1.2.1...
Installing RDoc documentation for bundler-1.2.1...
deadcow@jazz ~ $
```
e é isso


