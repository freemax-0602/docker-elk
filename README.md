Развертывание ELK-стека с помощью Ansible и Docker-compose

# 📌 Обзор
Проект автоматизирует развертывание стека ELK (Elasticsearch, Logstash, Kibana) и Nginx с Filebeat с использованием:
    - *Vagrant* для создания VM с тестовым стендом
    - *Ansible* для автоматизированного развертывания 
    - *Docker-compose* для оркестрации сервисов

# 🌟 Особенности
    - Польностью автоматизированное развертывание
    - Централизированный сбор логов через Filebeat → Logstash → Elasticsearch
    - Готовые дашборды в Kibana
    - Масштабируемая архитектура

# 🏗 Архитектура решения
Vagrant VM (web)                     Vagrant VM (elk)
┌──────────────────────┐             ┌──────────────────────┐
│  Контейнер Nginx     │             │  Elasticsearch       │
│  ┌───────────────┐   │  логи nginx │  ┌───────────────┐   │
│  │  Filebeat     ├───┼─────────────┼─►│  Logstash     │   │
│  └───────────────┘   │             │  └───────┬───────┘   │
└──────────────────────┘             │          │           │
                                     │  ┌───────▼───────┐   │
                                     │  │  Kibana       │   │
                                     │  └───────────────┘   │
                                     └──────────────────────┘


# 🚀 Быстрый старт
    1. Клонируйте репозиторий
    ```bash
    git clone https://github.com/ваш-репозиторий/elk-ansible-deployment.git
    cd elk-ansible-deployment
    ````
    2. Запустите виртуальные машины
    ```bash
    vagrant up
    ```
    3. Отредактируйте inventory-файл
    ```bash
    # Проброшенный в Vagrant VM 22 порт ssh просматривается командной vagrant ssh-config
    [service]
    web ansible_host=127.0.0.1 ansible_port=<port> ansible_private_key_file=../.vagrant/machines/web/virtualbox/private_key

    [logs]
    elk ansible_host=127.0.0.1 ansible_port=<port> ansible_private_key_file=../.vagrant/machines/elk/virtualbox/private_key
    ```
    4. Запусте Ansible playbook
    ```bash
    # Установка на VM docker и docker-compose
    ansible-playbook -i inventory deploy-vm/deploy-vm.yml
    # Установка веб-сервера nginx и filebeat на VM web 
    ansible-playbook -i inventory deploy-vm/deploy-nginx.yml 
    # Установка elk-стека на VM elk
    ansible-playbook -i inventory deploy-vm/deploy-elk.yml
    ```
# 🧪 Проверка работы
    1. Проверка запусков контйенеров nginx и filebeat на VM web
    ```bash
    # Проверка статуса контейнеров nginx и filebeat
    root@web:/opt/deploy-nginx$ sudo docker ps
    CONTAINER ID   IMAGE                     COMMAND                  CREATED        STATUS        PORTS                                            NAMES
    10d43ba53f59   elastic/filebeat:7.16.2   "sh -c 'chmod 644 /u…"   20 hours ago   Up 20 hours                                                    deploy-nginx-beats-1
    fb9126cf2a1e   nginx:1.10.1-alpine       "nginx -g 'daemon of…"   20 hours ago   Up 20 hours   443/tcp, 0.0.0.0:8080->80/tcp, :::8080->80/tcp   my-nginx
    # Проверка работы nginx
    root@web:/opt/deploy-nginx# curl http://localhost:8080
    <!doctype html>
    <html>
    <body style="background-color:rgb(49, 214, 220);"><center>
        <head>
        <title>Docker Project</title>
        </head>
        <body>
        <p>Welcome to my Docker Project!<p>
            <p>Today's Date and Time is: <span id='date-time'></span><p>
            <script>
                var dateAndTime = new Date();
                document.getElementById('date-time').innerHTML=dateAndTime.toLocaleString();
            </script>
            </body>
        </p>
    # Проверка работы filebeat
    root@web:/opt/deploy-nginx# docker logs -f 10d43 | tail -10
    2025-06-14T17:05:21.744Z	INFO	[file_watcher]	filestream/fswatch.go:137	Start next scan
    2025-06-14T17:05:31.740Z	INFO	[file_watcher]	filestream/fswatch.go:137	Start next scan
    2025-06-14T17:05:41.740Z	INFO	[file_watcher]	filestream/fswatch.go:137	Start next scan
    2025-06-14T17:05:51.738Z	INFO	[monitoring]	log/log.go:184	Non-zero metrics in the last 30s	{"monitoring": {"metrics": {"beat":{"cpu":{"system":{"ticks":3410,"time":{"ms":11}},"total":{"ticks":5670,"time":{"ms":11},"value":5670},"user":{"ticks":2260}},"handles":{"limit":{"hard":1048576,"soft":1048576},"open":12},"info":{"ephemeral_id":"88760c1d-8a44-4c7d-8cd1-96446b490bb4","uptime":{"ms":9120105},"version":"7.16.2"},"memstats":{"gc_next":18747600,"memory_alloc":11266952,"memory_total":238458760,"rss":118579200},"runtime":{"goroutines":33}},"filebeat":{"harvester":{"open_files":0,"running":0}},"libbeat":{"config":{"module":{"running":0}},"output":{"events":{"active":0}},"pipeline":{"clients":2,"events":{"active":0}}},"registrar":{"states":{"current":0}},"system":{"load":{"1":0.04,"15":0,"5":0.02,"norm":{"1":0.04,"15":0,"5":0.02}}}}}}
    2025-06-14T17:05:51.739Z	INFO	[file_watcher]	filestream/fswatch.go:137	Start next scan
    2025-06-14T17:06:01.739Z	INFO	[file_watcher]	filestream/fswatch.go:137	Start next scan
    ```
    2. Проверка запуска и состояния ELK на VM elk
    ```bash
    root@elk:/opt/deploy-elk# docker ps
    # Проверка запуска и состояния контейнеров ELK-стека на VM elk
    CONTAINER ID   IMAGE                  COMMAND                  CREATED        STATUS        PORTS                                                                                                                             NAMES
    023d8f279282   kibana:7.16.1          "/bin/tini -- /usr/l…"   20 hours ago   Up 20 hours   0.0.0.0:5601->5601/tcp, :::5601->5601/tcp                                                                                         deploy-elk-kibana-1
    d61de440a3d0   logstash:7.16.2        "/usr/local/bin/dock…"   20 hours ago   Up 20 hours   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp, 0.0.0.0:5044->5044/tcp, :::5044->5044/tcp, 0.0.0.0:9600->9600/tcp, :::9600->9600/tcp   deploy-elk-logstash-1
    b0547fbdaaff   elasticsearch:7.16.1   "/bin/tini -- /usr/l…"   20 hours ago   Up 20 hours   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp                                              deploy-elk-elasticsearch-1
    ```
    3. Запущенный веб-интерфейс Kibana (доступ по http://<your_IP>:5602)
    <img src="images/elasticsearch-status.jpg" width="600" alt="Elasticsearch Health Check">
    4. Индексы `nginx-logs-*` на созданном дашборде
     <img src="images/elasticsearch-status.jpg" width="600" alt="Index nginx-logs-* on Dashboard">
    
