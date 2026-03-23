# Настройка шардирования MongoDB

## 1. Запуск

```
docker compose up -d
```

### Инициализация Config Server
```
docker compose exec configdb mongosh --port 27019 --eval "
rs.initiate({
 _id: 'configrs',
 configsvr: true,
 members: [{ _id: 0, host: 'configdb:27019' }]
})"
```

### Инициализация шардов 1
```
docker compose exec shard1 mongosh --port 27018 --eval "
rs.initiate({
 _id: 'shard1rs',
 members: [{ _id: 0, host: 'shard1:27018' }]
})"
```

### Инициализация шардов 2
```
docker compose exec shard2 mongosh --port 27018 --eval "
rs.initiate({
 _id: 'shard2rs',
 members: [{ _id: 0, host: 'shard2:27018' }]
})"
```

### Подключение шардов к mongos
```
docker compose exec mongos mongosh --port 27017 --eval "
sh.addShard('shard1rs/shard1:27018');
sh.addShard('shard2rs/shard2:27018');
sh.enableSharding('somedb');
sh.shardCollection('somedb.users', { name: 'hashed' });
"
```
