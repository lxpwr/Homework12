</head>
<body lang="en-US" link="#000080" vlink="#800000" dir="ltr"><p style="line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="3" style="font-size: 14pt"><b><span style="background: transparent">1.
Запуск nginx на нестандартном порту 3
разными способами</span></b></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">По
умолчанию, SELinux не позволяет такой запуск
- служба не стартует при</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">запуске,
о чем свидетельствуют сообщения при
запуске ВМ:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">selinux:
Job for nginx.service failed because the control process exited with</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">error
code. See &quot;systemctl status nginx.service&quot; and &quot;journalctl
-xe&quot; for</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">details.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Сначала
потребовалось установить необходимые
для выполнения задания утилиты audit2why
audit2allow:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">yum
install setroubleshoot</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Их
не было в образе ВМ вагранта.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Утилита
audit2why позволяет узнать причину
неработоспособности службы:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Сначала
мне нужно было узнать, какое событие
анализировать с помощью этой </span></font></font></font>
</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">утилиты,
т.к. временной штамп в методичке, конечно
же другой.</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Для
этого я выполнил команду:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">grep
nginx /var/log/audit/audit.log</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">и
проанализировав вывод, обнаружил нужный
лог и обработал его:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">grep
1706686187.869:801 /var/log/audit/audit.log | audit2why</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><b><span style="background: transparent">Вывод
команды, собственно, и дает подсказку
для первого способа запуска nginx:</span></b></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Was
caused by:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">The
boolean nis_enabled was set incorrectly. </span></font></font></font>
</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Description:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Allow
nis to enabled</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Allow
access by executing:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">#
setsebool -P nis_enabled 1</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">После
выполнения setsebool -P nis_enabled 1 </span></font></font></font>
</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">nginx
благополучно запустился на порту 4881:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# systemctl restart nginx</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# systemctl status nginx</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">●
<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">nginx.service
- The nginx HTTP and reverse proxy server</span></font></font></span></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Loaded:
loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor
preset: disabled)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Active:
active (running) since Wed 2024-01-31 08:15:14 UTC; 10s ago</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3346 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3344 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3343 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited,
status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Main
PID: 3348 (nginx)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">CGroup:
/system.slice/nginx.service</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">├─<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">3348
nginx: master process /usr/sbin/nginx</span></font></font></span></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">└─<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">3350
nginx: worker process</span></font></font></span></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:15:14 selinux systemd[1]: Starting The nginx HTTP and reverse
proxy</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">server...</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:15:14 selinux nginx[3344]: nginx: the configuration file</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">/etc/nginx/nginx.conf
syntax is ok</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:15:14 selinux nginx[3344]: nginx: configuration file</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">/etc/nginx/nginx.conf
test is successful</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:15:14 selinux systemd[1]: Started The nginx HTTP and reverse
proxy</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">server.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><b><span style="background: transparent">Второй
способ запуска на нестандвртном порту
- включение этого порта в</span></b></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><b><span style="background: transparent">конфигурацию
SELinux с помощью команды semanage:</span></b></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# semanage port -l | grep http</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">http_cache_port_t
tcp 8080, 8118, 8123, 10001-10010</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">http_cache_port_t
udp 3130</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">http_port_t
tcp 80, 81, 443, 488, 8008, 8009, 8443, 9000</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">pegasus_http_port_t
tcp 5988</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">pegasus_https_port_t
tcp 5989</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Здесь
мы видим, что в парамете http_port_t нет
нужного нам 4881 и запуск будет</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">невозможен.</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Добавим
нужны порт:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# semanage port -a -t http_port_t -p tcp 4881</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">После
этого nginx опять радостно сообщает, что
у него все хорошо :) </span></font></font></font>
</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# systemctl restart nginx</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# systemctl status nginx</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">●
<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">nginx.service
- The nginx HTTP and reverse proxy server</span></font></font></span></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Loaded:
loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor
preset: disabled)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Active:
active (running) since Wed 2024-01-31 08:21:36 UTC; 3s ago</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3405 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3403 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3402 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited,
status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Main
PID: 3407 (nginx)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">CGroup:
/system.slice/nginx.service</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">├─<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">3407
nginx: master process /usr/sbin/nginx</span></font></font></span></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">└─<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">3409
nginx: worker process</span></font></font></span></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:21:36 selinux systemd[1]: Starting The nginx HTTP and reverse
proxy</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">server...</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:21:36 selinux nginx[3403]: nginx: the configuration file</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">/etc/nginx/nginx.conf
syntax is ok</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:21:36 selinux nginx[Ж3403]: nginx: configuration file</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">/etc/nginx/nginx.conf
test is successful</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:21:36 selinux systemd[1]: Started The nginx HTTP and reverse
proxy</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">server.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Видим,
что порт появился в списке:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# semanage port -l | grep http</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">http_cache_port_t
tcp 8080, 8118, 8123, 10001-10010</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">http_cache_port_t
udp 3130</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">http_port_t
tcp 4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">pegasus_http_port_t
tcp 5988</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">pegasus_https_port_t
tcp 5989</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Удаляем
его для проверки следующего способа:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# semanage port -d -t http_port_t -p tcp 4881</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><b><span style="background: transparent">Третий
способ - добавление модуля для nginx в
SELinux.</span></b></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">После
удаления порта nginx не запускается опять:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Jan
31 08:22:59 selinux systemd[1]: Failed to start The nginx HTTP and
reverse</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">proxy
server.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Теперь
используем другую утилиту audit2allow, чтобы
понять, как еще можно</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">заставить
работать nginx на 4881 порту. Для этого дадим
вывод лога утилите:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# grep nginx /var/log/audit/audit.log | audit2allow -M nginx</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">********************
IMPORTANT ***********************</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">To
make this policy package active, execute:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">semodule
-i nginx.pp</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Как
видим, ответ есть прямо в выводе. И введя
команду:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">semodule
-i nginx.pp</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">мы
опять получаем возможность запуска
сервера:</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# systemctl start nginx</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# systemctl status nginx</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">●
<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">nginx.service
- The nginx HTTP and reverse proxy server</span></font></font></span></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Loaded:
loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor
preset: disabled)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Active:
active (running) since Wed 2024-01-31 08:27:06 UTC; 3s ago</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3473 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3471 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Process:
3470 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited,
status=0/SUCCESS)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Main
PID: 3475 (nginx)</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">CGroup:
/system.slice/nginx.service</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">├─<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">3475
nginx: master process /usr/sbin/nginx</span></font></font></span></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><span style="background: transparent">└─<font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="font-weight: normal">3477
nginx: worker process</span></font></font></span></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Команда</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@selinux
~]# semodule -l | grep nginx</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">дает
нам:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">nginx
1.0</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">модуль
работает, сервер тоже :)</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="3" style="font-size: 14pt"><b><span style="background: transparent">2.Обеспечение
работоспособности приложения при
включенном SELinux</span></b></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Разворачиваем
стенд для работы.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Выполним
клонирование репозитория: </span></font></font></font>
</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">git
clone https://github.com/mbfx/otus-linux-adm.git</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">и
запускаем машины:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">vagrant
up</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Подключаюсь
к клиенту:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">vagrant
ssh client</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Ввод
команд по добавлению в зону DNS еще одной
записи не дает результата по</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">причине
неправильного контекста безопасности
для развернутой службы named:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">команда
для диагностики ошибок SELinux на клиенте
</span></font></font></font>
</p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">cat
/var/log/audit/audit.log | audit2why</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">возвращает
пустой вывод, что сообщает нам, что на
стороне клиента ошибок нет.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Диагностируем
сервер:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@ns01
~]# getenforce </span></font></font></font>
</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Enforcing</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@ns01
~]# cat /var/log/audit/audit.log | audit2allow</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0.2in"><br/>
<br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">#=============
named_t ==============</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">#!!!!
WARNING: 'etc_t' is a base type.</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">allow
named_t etc_t:file create;</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Сразу
видно, что контекст безопасности SELinux
неверный - вместо named_t у нас</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">etc_t</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Еще
подтверждение этому следующая команда:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@ns01
~]# ls -laZ /etc/named</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">drw-rwx---.
root named system_u:object_r:etc_t:s0 .</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">drwxr-xr-x.
root root system_u:object_r:etc_t:s0 ..</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">drw-rwx---.
root named unconfined_u:object_r:etc_t:s0 dynamic</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:etc_t:s0 named.50.168.192.rev</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:etc_t:s0 named.dns.lab</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:etc_t:s0 named.dns.lab.view1</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:etc_t:s0 named.newdns.lab</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Видно,
что установленный контекст - etc_t, а это
неправильно, изменения</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">конфигурации
невозможны</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Чтобы
понять, какой нам нужен правильный тип
контекста для работы, выполним его</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">поиск
с помоющью команды</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@ns01
~]# semanage fcontext -l | grep named</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Правильный
контекст - named_zone_t</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Теперь
нужно установить его на данный каталог
- тогда изменения конфигурации</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">будут
возможны - </span></font></font></font>
</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">это
самый простой способ заставить работать
сервис DNS. Нам не придется</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">выполнять
переустановку или менятькаталоги
приложения.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Меняем
контекст на верный:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@ns01
~]# sudo chcon -R -t named_zone_t /etc/named</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">и
проверяем:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">[root@ns01
~]# ls -laZ /etc/named</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">drw-rwx---.
root named system_u:object_r:named_zone_t:s0 .</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">drwxr-xr-x.
root root system_u:object_r:etc_t:s0 ..</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">drw-rwx---.
root named unconfined_u:object_r:named_zone_t:s0 dynamic</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.50.168.192.rev</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.dns.lab</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.dns.lab.view1</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-rw-rw----.
root named system_u:object_r:named_zone_t:s0 named.newdns.lab</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">После
этих манипуляций изменения зоны с
клиента проходит без ошибок, все.</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">Для
того, чтобы стенд был рабочим изначально,
я модифицировал playbook.yml</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">добавив
в него секцию с выполнением команды
смены контекста на сервере ns01:</span></font></font></font></p>
<p style="line-height: 0.2in; margin-bottom: 0in"><br/>

</p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">-
name: fixing SELinux context</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">ansible.builtin.command:
/usr/bin/chcon -R -t named_zone_t /etc/named</span></font></font></font></p>
<p style="font-weight: normal; line-height: 0.2in; margin-bottom: 0in">
<font color="#000000"><font face="Droid Sans Mono, monospace, monospace"><font size="2" style="font-size: 10pt"><span style="background: transparent">После
этого стенд загружается и изменение
зоны возможно сразу.</span></font></font></font></p>
<p style="line-height: 100%; margin-bottom: 0in"><br/>

</p>
</body>
</html>
