Instalar GitLab Enterprise Edition
===================================

https://docs.gitlab.com/omnibus/README.html#installation-and-configuration-using-omnibus-package

Instalamos este pre requisito en Centos 7.::

	# yum install policycoreutils-python.x86_64

Descargamos el Gitlab puede tambien ir a la pagina y buscar la versión que usted necesita.::

	# wget https://packages.gitlab.com/gitlab/gitlab-ee/packages/el/7/gitlab-ee-11.2.3-ee.0.el7.x86_64.rpm

Instalamos el Gitlab.::

	# EXTERNAL_URL="www.dominio.local" # rpm -ivh gitlab-ee-11.2.3-ee.0.el7.x86_64.rpm

	[...]
	Thank you for installing GitLab!
	GitLab was unable to detect a valid hostname for your instance.
	Please configure a URL for your GitLab instance by setting `external_url`
	configuration in /etc/gitlab/gitlab.rb file.
	Then, you can start your GitLab instance by running the following command:
	  sudo gitlab-ctl reconfigure

	For a comprehensive list of configuration options please see the Omnibus GitLab readme
	https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

Procedemos a a ejecutar la autoconfiguración de Gitlab.::

	gitlab-ctl reconfigure
	[...]
	Running handlers complete
	Chef Client finished, 429/638 resources updated in 03 minutes 16 seconds
	gitlab Reconfigured!


Verificamos el status.::

	# gitlab-ctl status
	run: alertmanager: (pid 5798) 14s; run: log: (pid 5808) 14s
	run: gitaly: (pid 5741) 17s; run: log: (pid 5752) 16s
	run: gitlab-monitor: (pid 5760) 16s; run: log: (pid 5765) 16s
	run: gitlab-workhorse: (pid 5726) 17s; run: log: (pid 5734) 17s
	run: logrotate: (pid 5285) 108s; run: log: (pid 5733) 17s
	run: nginx: (pid 5963) 1s; run: log: (pid 5732) 17s
	run: node-exporter: (pid 5362) 95s; run: log: (pid 5751) 16s
	run: postgres-exporter: (pid 5815) 14s; run: log: (pid 5822) 14s
	run: postgresql: (pid 5003) 178s; run: log: (pid 5718) 17s
	run: prometheus: (pid 5781) 15s; run: log: (pid 5787) 15s
	run: redis: (pid 4943) 190s; run: log: (pid 5719) 17s
	run: redis-exporter: (pid 5478) 83s; run: log: (pid 5764) 16s
	run: sidekiq: (pid 5245) 121s; run: log: (pid 5717) 17s
	run: unicorn: (pid 5208) 127s; run: log: (pid 5716) 17s


Detenemos el servicio.::

	# gitlab-ctl stop
	ok: down: alertmanager: 1s, normally up
	ok: down: gitaly: 0s, normally up
	ok: down: gitlab-monitor: 0s, normally up
	ok: down: gitlab-workhorse: 1s, normally up
	ok: down: logrotate: 0s, normally up
	ok: down: nginx: 1s, normally up
	ok: down: node-exporter: 0s, normally up
	ok: down: postgres-exporter: 1s, normally up
	ok: down: postgresql: 0s, normally up
	ok: down: prometheus: 0s, normally up
	ok: down: redis: 0s, normally up
	ok: down: redis-exporter: 1s, normally up
	ok: down: sidekiq: 1s, normally up


Iniciamos el servicio.::

	# gitlab-ctl start
	ok: run: alertmanager: (pid 6126) 1s
	ok: run: gitaly: (pid 6134) 0s
	ok: run: gitlab-monitor: (pid 6145) 1s
	ok: run: gitlab-workhorse: (pid 6156) 0s
	ok: run: logrotate: (pid 6163) 0s
	ok: run: nginx: (pid 6172) 1s
	ok: run: node-exporter: (pid 6174) 0s
	ok: run: postgres-exporter: (pid 6178) 1s
	ok: run: postgresql: (pid 6183) 0s
	ok: run: prometheus: (pid 6185) 1s
	ok: run: redis: (pid 6199) 0s
	ok: run: redis-exporter: (pid 6242) 1s
	ok: run: sidekiq: (pid 6285) 0s


Verificamos todos los puertos que coloca en listen.::

	# netstat -natp | grep -i listen
	tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      2211/mysqld         
	tcp        0      0 127.0.0.1:9100          0.0.0.0:*               LISTEN      7097/node_exporter  
	tcp        0      0 127.0.0.1:9229          0.0.0.0:*               LISTEN      7076/gitlab-workhor 
	tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      7092/nginx: master  
	tcp        0      0 127.0.0.1:9168          0.0.0.0:*               LISTEN      7065/ruby           
	tcp        0      0 127.0.0.1:9236          0.0.0.0:*               LISTEN      7054/gitaly         
	tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      827/sshd            
	tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      2177/master         
	tcp        0      0 0.0.0.0:8060            0.0.0.0:*               LISTEN      7092/nginx: master  
	tcp        0      0 127.0.0.1:9121          0.0.0.0:*               LISTEN      7127/redis_exporter 
	tcp        0      0 127.0.0.1:9090          0.0.0.0:*               LISTEN      7114/prometheus     
	tcp        0      0 127.0.0.1:9187          0.0.0.0:*               LISTEN      7101/postgres_expor 
	tcp        0      0 127.0.0.1:9093          0.0.0.0:*               LISTEN      7046/alertmanager   
	tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      2326/slapd          
	tcp6       0      0 ::1:9168                :::*                    LISTEN      7065/ruby           
	tcp6       0      0 :::22                   :::*                    LISTEN      827/sshd            
	tcp6       0      0 ::1:25                  :::*                    LISTEN      2177/master         
	tcp6       0      0 :::389                  :::*                    LISTEN      2326/slapd          
	tcp6       0      0 :::9094                 :::*                    LISTEN      7046/alertmanager    


Verificamos y más que todo para tener un histórico de todos los procesos que deben levantar.::

	# ps -ef | grep git
	root      4921     1  0 17:09 ?        00:00:00 runsvdir -P /opt/gitlab/service log: ...........................................................................................................................................................................................................................................................................................................................................................................................................
	root      5261  4921  0 17:11 ?        00:00:00 runsv gitlab-workhorse
	root      5304  4921  0 17:11 ?        00:00:00 runsv gitaly
	root      5455  4921  0 17:11 ?        00:00:00 runsv gitlab-monitor
	root      5716  5206  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/unicorn
	root      5717  5243  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/sidekiq
	root      5718  5001  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/postgresql
	root      5719  4941  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/redis
	root      5732  5268  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/nginx
	root      5733  5283  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/logrotate
	root      5734  5261  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/gitlab-workhorse
	root      5751  5360  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/node-exporter
	root      5752  5304  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/gitaly
	root      5764  5476  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/redis-exporter
	root      5765  5455  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/gitlab-monitor
	root      5787  5494  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/prometheus
	root      5808  5524  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/alertmanager
	root      5822  5562  0 17:12 ?        00:00:00 svlogd -tt /var/log/gitlab/postgres-exporter
	gitlab-+  7046  5524  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/alertmanager --web.listen-address=localhost:9093 --storage.path=/var/opt/gitlab/alertmanager/data --config.file=/var/opt/gitlab/alertmanager/alertmanager.yml
	git       7054  5304  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/gitaly /var/opt/gitlab/gitaly/config.toml
	git       7065  5455  1 17:18 ?        00:00:01 /opt/gitlab/embedded/bin/ruby /opt/gitlab/embedded/bin/gitlab-mon web -c /var/opt/gitlab/gitlab-monitor/gitlab-monitor.yml
	git       7067  7054  1 17:18 ?        00:00:01 ruby /opt/gitlab/embedded/service/gitaly-ruby/bin/gitaly-ruby 7054 /tmp/gitaly-ruby955959495/socket.0
	git       7069  7054  1 17:18 ?        00:00:01 ruby /opt/gitlab/embedded/service/gitaly-ruby/bin/gitaly-ruby 7054 /tmp/gitaly-ruby955959495/socket.1
	git       7076  5261  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/gitlab-workhorse -listenNetwork unix -listenUmask 0 -listenAddr /var/opt/gitlab/gitlab-workhorse/socket -authBackend http://localhost:8080 -authSocket /var/opt/gitlab/gitlab-rails/sockets/gitlab.socket -documentRoot /opt/gitlab/embedded/service/gitlab-rails/public -pprofListenAddr  -prometheusListenAddr localhost:9229 -secretPath /opt/gitlab/embedded/service/gitlab-rails/.gitlab_workhorse_secret -config config.toml
	root      7083  5283  0 17:18 ?        00:00:00 /bin/sh /opt/gitlab/embedded/bin/gitlab-logrotate-wrapper
	root      7092  5268  0 17:18 ?        00:00:00 nginx: master process /opt/gitlab/embedded/sbin/nginx -p /var/opt/gitlab/nginx
	gitlab-+  7093  7092  0 17:18 ?        00:00:00 nginx: worker process
	gitlab-+  7094  7092  0 17:18 ?        00:00:00 nginx: cache manager process
	gitlab-+  7097  5360  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/node_exporter --web.listen-address=localhost:9100 --collector.textfile.directory=/var/opt/gitlab/node-exporter/textfile_collector
	gitlab-+  7101  5562  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/postgres_exporter --web.listen-address=localhost:9187 --extend.query-path=/var/opt/gitlab/postgres-exporter/queries.yaml
	gitlab-+  7106  5001  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/postgres -D /var/opt/gitlab/postgresql/data
	gitlab-+  7108  7106  0 17:18 ?        00:00:00 postgres: checkpointer process   
	gitlab-+  7109  7106  0 17:18 ?        00:00:00 postgres: writer process   
	gitlab-+  7110  7106  0 17:18 ?        00:00:00 postgres: wal writer process   
	gitlab-+  7111  7106  0 17:18 ?        00:00:00 postgres: autovacuum launcher process   
	gitlab-+  7112  7106  0 17:18 ?        00:00:00 postgres: stats collector process   
	gitlab-+  7114  5494  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/prometheus -web.listen-address=localhost:9090 -storage.local.path=/var/opt/gitlab/prometheus/data -storage.local.chunk-encoding-version=2 -storage.local.target-heap-size=105728163 -config.file=/var/opt/gitlab/prometheus/prometheus.yml
	gitlab-+  7122  4941  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/redis-server 127.0.0.1:0
	gitlab-+  7127  5476  0 17:18 ?        00:00:00 /opt/gitlab/embedded/bin/redis_exporter -web.listen-address=localhost:9121 -redis.addr=unix:///var/opt/gitlab/redis/redis.socket
	git       7206  5243 26 17:18 ?        00:00:36 sidekiq 5.1.3 gitlab-rails [0 of 25 busy]
	git       7213  5206  0 17:18 ?        00:00:00 /bin/bash /opt/gitlab/embedded/bin/gitlab-unicorn-wrapper
	git       7234     1 27 17:18 ?        00:00:38 unicorn master -D -E production -c /var/opt/gitlab/gitlab-rails/etc/unicorn.rb /opt/gitlab/embedded/service/gitlab-rails/config.ru
	gitlab-+  7237  7106  0 17:18 ?        00:00:00 postgres: gitlab-psql postgres [local] idle
	gitlab-+  7240  7106  0 17:18 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
	gitlab-+  7241  7106  0 17:18 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
	gitlab-+  7242  7106  0 17:18 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
	git       7359  7234  1 17:20 ?        00:00:00 unicorn worker[0] -D -E production -c /var/opt/gitlab/gitlab-rails/etc/unicorn.rb /opt/gitlab/embedded/service/gitlab-rails/config.ru
	git       7362  7234  2 17:20 ?        00:00:01 unicorn worker[1] -D -E production -c /var/opt/gitlab/gitlab-rails/etc/unicorn.rb /opt/gitlab/embedded/service/gitlab-rails/config.ru
	gitlab-+  7389  7106  0 17:20 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
	gitlab-+  7424  7106  0 17:20 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
	gitlab-+  7426  7106  0 17:20 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
	gitlab-+  7476  7106  0 17:20 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
	git       8820  7213  0 17:21 ?        00:00:00 sleep 1
	root      8822  2536  0 17:21 pts/0    00:00:00 grep --color=auto git


Listo...!!! ahora por favor visualice el documento de Configuración inicial







