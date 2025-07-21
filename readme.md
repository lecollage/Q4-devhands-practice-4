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

## Publish/Subscribe
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

