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
docker compose exec shard1a mongosh --port 27018 --eval "
    rs.initiate({
      _id: 'shard1rs',
      members: [
        { _id: 0, host: 'shard1a:27018' },
        { _id: 1, host: 'shard1b:27018' },
        { _id: 2, host: 'shard1c:27018' }
      ]
    })"
```

### Инициализация шардов 2
```
docker compose exec shard2a mongosh --port 27018 --eval "
    rs.initiate({
      _id: 'shard2rs',
      members: [
        { _id: 0, host: 'shard2a:27018' },
        { _id: 1, host: 'shard2b:27018' },
        { _id: 2, host: 'shard2c:27018' }
      ]
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
