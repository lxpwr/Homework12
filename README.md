<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="content-type" content="text/html; charset=utf-8"/>
	<title></title>
	<meta name="generator" content="LibreOffice 7.3.7.2 (Linux)"/>
	<meta name="created" content="00:00:00"/>
	<meta name="changed" content="2024-01-31T14:57:28.631359646"/>
	<style type="text/css">
		@page { size: 8.27in 11.69in; margin: 0.79in }
		p { line-height: 115%; margin-bottom: 0.1in; background: transparent; background: transparent }
		pre { background: transparent; background: transparent }
		pre.western { font-family: "Liberation Mono", monospace; font-size: 10pt }
		pre.cjk { font-family: "Noto Sans Mono CJK SC", monospace; font-size: 10pt }
		pre.ctl { font-family: "Liberation Mono", monospace; font-size: 10pt }
		a:link { color: #000080; text-decoration: underline }
		a:visited { color: #800000; text-decoration: underline }
	</style>
</head>
<body lang="en-US" link="#000080" vlink="#800000" dir="ltr"><pre class="western"><font size="3" style="font-size: 12pt"><b>1. </b><b>Запуск nginx на нестандвртном порту 3 разными способами</b></font>

По умолчанию, SELinux не позволяет такой запуск - служба не стартует при
 запуске, о чем свидетельствуют сообщения при запуске ВМ:

  selinux: Job for nginx.service failed because the control process exited with
 error code. See &quot;systemctl status nginx.service&quot; and &quot;journalctl -xe&quot; for
 details.

  
Сначала потребовалось установить необходимые для выполнения задания утилиты audit2why audit2allow:

yum install setroubleshoot

Их не было в образе ВМ вагранта.

Утилита audit2why позволяет узнать причину неработоспособности службы:

Сначала мне нужно было узнать, какое событие анализировать с помощью этой 
утилиты, т.к. временной штамп в методичке, конечно же другой.
Для этого я выполнил команду:
grep nginx /var/log/audit/audit.log
и проанализировав вывод, обнаружил нужный лог и обработал его:

grep 1706686187.869:801 /var/log/audit/audit.log | audit2why

<b>Вывод команды, собственно, и дает подсказку для первого способа запуска nginx:</b>

Was caused by:
        The boolean nis_enabled was set incorrectly. 
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1

После выполнения setsebool -P nis_enabled 1 

nginx благополучно запустился на порту 4881:

[root@selinux ~]# systemctl restart  nginx
[root@selinux ~]# systemctl status  nginx

● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-01-31 08:15:14 UTC; 10s ago
  Process: 3346 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3344 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 3343 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 3348 (nginx)
   CGroup: /system.slice/nginx.service
           ├─3348 nginx: master process /usr/sbin/nginx
           └─3350 nginx: worker process

Jan 31 08:15:14 selinux systemd[1]: Starting The nginx HTTP and reverse proxy
 server...
Jan 31 08:15:14 selinux nginx[3344]: nginx: the configuration file
 /etc/nginx/nginx.conf syntax is ok
Jan 31 08:15:14 selinux nginx[3344]: nginx: configuration file
 /etc/nginx/nginx.conf test is successful
Jan 31 08:15:14 selinux systemd[1]: Started The nginx HTTP and reverse proxy
 server.

<b>Второй способ запуска на нестандвртном порту - включение этого порта в</b>
 <b>конфигурацию SELinux с помощью команды semanage:</b>

[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989

Здесь мы видим, что в парамете http_port_t нет нужного нам 4881 и запуск будет
 невозможен.
Добавим нужны порт:

[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881

После этого nginx опять радостно сообщает, что у него все хорошо :) 

[root@selinux ~]# systemctl restart  nginx
[root@selinux ~]# systemctl status  nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-01-31 08:21:36 UTC; 3s ago
  Process: 3405 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3403 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 3402 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 3407 (nginx)
   CGroup: /system.slice/nginx.service
           ├─3407 nginx: master process /usr/sbin/nginx
           └─3409 nginx: worker process

Jan 31 08:21:36 selinux systemd[1]: Starting The nginx HTTP and reverse proxy
 server...
Jan 31 08:21:36 selinux nginx[3403]: nginx: the configuration file
 /etc/nginx/nginx.conf syntax is ok
Jan 31 08:21:36 selinux nginx[Ж3403]: nginx: configuration file
 /etc/nginx/nginx.conf test is successful
Jan 31 08:21:36 selinux systemd[1]: Started The nginx HTTP and reverse proxy
 server.

Видим, что порт появился в списке:

[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989

Удаляем его для проверки следующего способа:

[root@selinux ~]# semanage port -d -t http_port_t -p tcp 4881

<b>Третий способ - добавление модуля для nginx в SELinux.</b>

После удаления порта nginx не запускается опять:
Jan 31 08:22:59 selinux systemd[1]: Failed to start The nginx HTTP and reverse
 proxy server.

Теперь используем другую утилиту audit2allow, чтобы понять, как еще можно
 заставить работать nginx на 4881 порту. Для этого дадим вывод лога утилите:
[root@selinux ~]# grep nginx /var/log/audit/audit.log | audit2allow -M nginx

******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp

Как видим, ответ есть прямо в выводе. И введя команду:
semodule -i nginx.pp
мы опять получаем возможность запуска сервера:
[root@selinux ~]# systemctl start  nginx
[root@selinux ~]# systemctl status  nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-01-31 08:27:06 UTC; 3s ago
  Process: 3473 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3471 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 3470 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 3475 (nginx)
   CGroup: /system.slice/nginx.service
           ├─3475 nginx: master process /usr/sbin/nginx
           └─3477 nginx: worker process

Команда
[root@selinux ~]# semodule -l | grep nginx

дает нам:

nginx   1.0

модуль работает, сервер тоже :)
</pre><p style="font-variant: normal; font-style: normal; line-height: 100%; margin-bottom: 0.2in; background: transparent; text-decoration: none">
<a name="docs-internal-guid-8d29ed9b-7fff-cbab-2a4d-263636fe2ffe"></a>
<font color="#000000"><font face="Courier New, monospace"><font size="3" style="font-size: 12pt"><b><span style="background: transparent">2.Обеспечение
работоспособности приложения при
включенном SELinux</span></b></font></font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Разворачиваем
стенд для работы.</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Выполним
клонирование репозитория: </font></font>
</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">git
clone <a href="https://github.com/mbfx/otus-linux-adm.git">https://github.com/mbfx/otus-linux-adm.git</a></font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">и
запускаем машины:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">vagrant
up</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Подключаюсь
к клиенту:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">vagrant
ssh client</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Ввод
команд по добавлению в зону DNS еще одной
записи не дает результата по</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
 <font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">причине
неправильного контекста безопасности
для развернутой службы named:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">команда
для диагностики ошибок SELinux на клиенте
</font></font>
</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">cat
/var/log/audit/audit.log | audit2why</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">возвращает
пустой вывод, что сообщает нам, что на
стороне клиента ошибок нет.</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Диагностируем
сервер:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">[root@ns01
~]# getenforce </font></font>
</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Enforcing</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">[root@ns01
~]# cat /var/log/audit/audit.log | audit2allow</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">#=============
named_t ==============</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">#!!!!
WARNING: 'etc_t' is a base type.</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">allow
named_t etc_t:file create;</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Сразу
видно, что контекст безопасности SELinux
неверный - вместо named_t у нас</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
 <font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">etc_t</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Еще
подтверждение этому следующая команда:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">[root@ns01
~]# ls -laZ /etc/named</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">drw-rwx---.
root named system_u:object_r:etc_t:s0       .</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">drwxr-xr-x.
root root  system_u:object_r:etc_t:s0       ..</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">drw-rwx---.
root named unconfined_u:object_r:etc_t:s0   dynamic</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:etc_t:s0       named.50.168.192.rev</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:etc_t:s0       named.dns.lab</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:etc_t:s0       named.dns.lab.view1</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:etc_t:s0       named.newdns.lab</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Видно,
что установленный контекст - etc_t, а это
неправильно, изменения</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
 <font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">конфигурации
невозможны</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Чтобы
понять, какой нам нужен правильный тип
контекста для работы, выполним его</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
 <font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">поиск
с помоющью команды</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">[root@ns01
~]# semanage fcontext -l | grep named</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Правильный
контекст - named_zone_t</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Теперь
нужно установить его на данный каталог
- тогда изменения конфигурации</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
 <font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">будут
возможны - </font></font>
</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">это
самый простой способ заставить работать
сервис DNS. Нам не придется</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
 <font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">выполнять
переустановку или менятькаталоги
приложения.</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Меняем
контекст на верный:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">[root@ns01
~]# sudo chcon -R -t named_zone_t /etc/named</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">и
проверяем:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">[root@ns01
~]# ls -laZ /etc/named</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">drw-rwx---.
root named system_u:object_r:named_zone_t:s0 .</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">drwxr-xr-x.
root root  system_u:object_r:etc_t:s0       ..</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">drw-rwx---.
root named unconfined_u:object_r:named_zone_t:s0 dynamic</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.50.168.192.rev</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.dns.lab</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.dns.lab.view1</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.newdns.lab</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">После
этих манипуляций изменения зоны с
клиента проходит без ошибок, все.</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">Для
того, чтобы стенд был рабочим изначально,
я модифицировал playbook.yml</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">добавив
в него секцию с выполнением команды
смены контекста на сервере ns01:</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<br/>

</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">-
name: fixing SELinux context</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
    <font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">ansible.builtin.command:
/usr/bin/chcon -R -t named_zone_t /etc/named</font></font></p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
      
</p>
<p style="line-height: 100%; margin-bottom: 0in; background: transparent">
<font face="Liberation Mono, monospace"><font size="2" style="font-size: 10pt">После
этого стенд загружается и изменение
зоны возможно сразу.</font></font></p>
</body>
</html>