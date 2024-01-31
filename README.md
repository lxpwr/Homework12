<body lang="en-US" link="#000080" vlink="#800000" dir="ltr"><pre class="western">Запуск nginx на нестандвртном порту 3 разными способами

По умолчанию, SELinux не позволяет такой запуск - служба не стартует при запуске, о чем свидетельствуют 
сообщения при запуске ВМ:

  selinux: Job for nginx.service failed because the control process exited with error code. 
  See &quot;systemctl status nginx.service&quot; and &quot;journalctl -xe&quot; for details.

  
Сначала потребовалось установить необходимые для выполнения задания утилиты audit2why audit2allow:

yum install setroubleshoot

Их не было в образе ВМ вагранта.

Утилита audit2why позволяет узнать причину неработоспособности службы:

Сначала мне нужно было узнать, какое событие анализировать с помощью этой утилиты, т.к. временной штамп в методичке, 
конечно же другой.
Для этого я выполнил команду:
grep nginx /var/log/audit/audit.log
и проанализировав вывод, обнаружил нужный лог и обработал его:

grep 1706686187.869:801 /var/log/audit/audit.log | audit2why

Вывод команды, собственно, и дает подсказку для первого способа запуска nginx:

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

Jan 31 08:15:14 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 31 08:15:14 selinux nginx[3344]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 31 08:15:14 selinux nginx[3344]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 31 08:15:14 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.

Второй способ запуска на нестандвртном порту - включение этого порта в конфигурацию SELinux с помощью команды semanage:

[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989

Здесь мы видим, что в парамете http_port_t нет нужного нам 4881 и запуск будет невозможен.
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

Jan 31 08:21:36 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 31 08:21:36 selinux nginx[3403]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 31 08:21:36 selinux nginx[Ж3403]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 31 08:21:36 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.

Видим, что порт появился в списке:

[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989

Удаляем его для проверки следующего способа:

[root@selinux ~]# semanage port -d -t http_port_t -p tcp 4881

Третий способ - добавление модуля для nginx в SELinux.

После удаления порта nginx не запускается опять:
Jan 31 08:22:59 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.

Теперь используем другую утилиту audit2allow, чтобы понять, как еще можно заставить работать nginx на 4881 порту. 
Для этого дадим вывод лога утилите:

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

nginx	1.0

модуль работает, сервер тоже :)</pre>
</body>
</html>
