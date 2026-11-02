# Задача 1: API Gateway

| Критерий | Nginx | Kong | Traefik | Apache APISIX | Spring Cloud Gateway |
|---|---|---|---|---|---|
| **Маршрутизация по конфигурации** | + | + | + | + | + |
| **Аутентификация** | Basic Auth, JWT (только NGINX Plus; в OSS — через сторонние модули) | Basic Auth, JWT, Key Auth, HMAC, OAuth 2.0, OIDC, mTLS, LDAP | Basic Auth, Digest Auth, ForwardAuth | Basic Auth, JWT, Key Auth, HMAC, OAuth 2.0, OIDC, mTLS, LDAP | Через Spring Security (фильтры JWT/Basic и др. встроены, полная логика — на Spring Security) |
| **Терминация HTTPS** | + | + | + | + | + |
| **Открытость** | Есть Enterprise-версия, но базовой должно хватить для большинства задач | Есть Enterprise-версия, но базовой должно хватить для большинства задач | Есть Enterprise-версия, но базовой должно хватить для большинства задач | Полностью открытый | Полностью открытый |
| **Динамическая конфигурация** | − | + | + | + | + (через Spring Cloud Config Server / Actuator, требуется внешний триггер обновления) |
| **Мониторинг** | Минимальный (`stub_status`) | Prometheus-плагин, healthcheck для upstream | Встроенный `/metrics`, healthcheck для upstream | Встроенный `/metrics`, healthcheck для upstream | Встроенный `/metrics`, healthcheck для upstream |
| **Модульность / расширяемость** | Доп. модули через разработку на C и Lua, высокий порог входа | Огромное количество плагинов | Экосистема плагинов есть, но скуднее, чем у конкурентов | Огромное количество плагинов | Отдельного репозитория плагинов нет; расширение через фильтры (встроенные в конфигурации или кастомные на Java) |


В результате я бы рекомендовал использовать Kong или Apache APISIX, т.к. в целом они одинаково хороши по характеристикам.


# Задача 2: Брокер сообщений


| Критерий | Apache Kafka | RabbitMQ | Redis Streams | Apache Pulsar | NATS |
|---|---|---|---|---|---|
| **Поддержка кластеризации** | + | + | + | + | + |
| **Хранение сообщений на диске** | + | + | ± (опционально, слабее остальных) | + | + |
| **Пропускная способность** | Очень высокая | Средняя | Высокая / Очень высокая (in-memory) | Очень высокая | Очень высокая |
| **Задержка** | Низкая | Очень низкая | Очень низкая | Низкая | Очень низкая |
| **Поддержка различных форматов сообщений** | Высокая (любые байты + Schema Registry; ограничение по протоколу) | Отличная (AMQP, MQTT, STOMP и др.) | Низкая (пары ключ-значение, формат задаёт приложение) | Средняя (мультипротокол, совместим с Kafka/RabbitMQ) | Средняя (собственный протокол, заголовки и полезная нагрузка произвольные, но нет нативной поддержки AMQP/MQTT) |
| **Разделение прав доступа к потокам** | Средняя (SSL/SASL, ACL) | Высокая (Vhost + User, гибкие права) | Низкая (ACL по ключам, без детализации ыпо группам потребителей) | Высокая (мультиарендность, namespace, токены) | Высокая (аккаунты + JWT) |
| **Простота эксплуатации** | Средняя (зависимость от KRaft, меньше встроенных инструментов) | Высокая (встроенная консоль, прост в освоении) | Высокая | Низкая (самый сложный из приведённых) | Высокая (один бинарник, минимальная конфигурация) |


При равнозначности всех шести критериев Apache Pulsar остаётся наиболее сбалансированным решением. Если приоритет - простота эксплуатации и скорость, выбираем NATS JetStream. Если нужна максимальная пропускная способность и зрелая экосистема - Apache Kafka. Если критичны гибкость протоколов и простота - RabbitMQ.


# Задача 3: API Gateway * (необязательная)
Был разработан прототип API Gateway на базе nginx, конфигурация представлена в [файле](11-microservices-02-principles/gateway/nginx.conf).

Команды для тестирования решения:

1. `curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/v1/token`
2. `curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @cat.jpg http://localhost/v1/upload`
3. `curl -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' http://localhost/v1/user/{image-id}.jpg --output result.jpg`

Пример тестирования:

![Скриншот 1](images/1.png)