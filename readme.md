# Q04 // Queues - Devhands.io // NATS Practice 4 

## Run NATS cluster
```bash
docker-compose up
```

## Prepare 
```bash
docker-compose exec cli bash # or alias nats "docker-compose exec cli nats"

nats --completion-script-bash >> ~/.bashrc; source ~/.bashrc

nats context add sys --server ru-1:4222,ru-2:4222,ru-3:4222 --user admin --password Pswd1
nats context add ru --server ru-1:4222,ru-2:4222,ru-3:4222 --user user --password Pswd1 --select

nats server ping --context sys

nats server ls --sort name --context sys
```

## Perform the benchmark
```bash 
docker-compose exec cli bash
nats bench sub test-subject
```
```bash 
docker-compose exec cli bash
nats bench pub test-subject
```

## Basic scenarios

### Publish/Subscribe
```bash 
docker-compose exec cli bash
nats sub 'topic.>'
```
```bash 
docker-compose exec cli bash

nats pub topic test-message

nats pub topic.a test-a
nats pub topic.b test-b
nats pub topic.c.d test-c-d

nats pub topic.a test-a
nats pub topic.b test-b
nats pub topic.c.d test-c-d
```

### Work queue (consumer group)
```bash
docker-compose exec cli bash
nats sub topic --queue grp1
```
```bash
docker-compose exec cli bash
nats pub topic --count 10 '{{ID}} : {{Count}} : {{Random 5 5}}'
```

### Request/Response
```bash
docker-compose exec cli bash
nats reply topic 'RE: {{ Request }}'
```
```bash
nats sub '>'
```
```bash
docker-compose exec cli bash
nats pub topic test-message
nats request topic test-request

nats pub topic test --reply=topic-re
nats pub topic --count 10 '{{ID}} : {{Count}} : {{Random 5 5}}' --reply=topic-re
```

## Request/Response supercluster scenario
```bash
docker-compose exec cli bash
nats context add eu --server eu-1:4222,eu-2:4222,eu-3:4222 --user user --password Pswd1
nats context add as --server as-1:4222,as-2:4222,as-3:4222 --user user --password Pswd1
```

```bash
docker-compose exec cli bash
nats reply echo '[RU] Получено: {{ Request }}' --context ru
```

```bash
docker-compose exec cli bash
nats reply echo '[EU] Received: {{ Request }}' --context eu
```

```bash
docker-compose exec cli bash
nats reply echo '[AS] 수신됨: {{ Request }}' --context as
```

```bash
docker-compose exec cli bash
nats request echo "Msg {{Random 15 15}}" --context ru
nats request echo "Msg {{Random 15 15}}" --context eu
nats request echo "Msg {{Random 15 15}}" --context as
```

## Streams
```bash
docker-compose exec cli bash
nats stream add DURABLE --subjects "durable.>"
```
All options should have default values

```bash
nats consumer add
```
All options should have default values

```bash
docker-compose exec cli bash
nats pub durable.a "message 1"
nats pub durable.a "message 2"
```

```bash
docker-compose exec cli bash
nats consumer info DURABLE CONS
nats consumer update DURABLE CONS --wait 10s
```

```bash
nats consumer next DURABLE CONS
nats consumer next DURABLE CONS --no-ack
nats consumer next DURABLE CONS --no-ack
nats consumer next DURABLE CONS
nats consumer next DURABLE CONS
```

## NATS Key/Value

```bash
docker-compose exec cli bash

nats kv add configs --history=5 --description "App configs" --replicas 3
nats kv put configs db.host localhost
nats kv get configs db.host
nats kv put configs db.host db.prod.internal
nats kv history configs db.host
```

```bash
docker-compose exec cli bash
nats kv put configs db.host db.failover.internal
nats kv watch configs db.host
```

```bash
docker-compose exec cli bash
nats kv del configs db.host
nats kv purge configs db.host
```

## NATS Object Store
```bash
dd if=/dev/urandom of=backup.dump bs=10M count=1 && gzip backup.dump
```

```bash
nats obj add backups --description "Daily DB backups" --storage file --replicas 3
nats obj put backups ./backup.dump.gz --name backup-2025-04-22.tgz
nats obj ls backups
nats obj get backups backup-2025-04-22.tgz
nats obj info backups backup-2025-04-22.tgz
nats obj del backups backup-2025-04-22.tgz
nats obj seal backups
nats obj del backups
```