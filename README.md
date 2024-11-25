# mongo-sharding

# Запускаем

docker compose up -d


# Инициализация сервера конфигурации

docker exec -it configSrv mongosh --port 27020

rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27020" }
    ]
  }
);

exit();

# Инициализация шардов

docker exec -it shard1-1 mongosh --port 27021

rs.initiate(
    {
      _id : "shard1-1",
      members: [
        { _id : 0, host : "shard1-1:27021" }
      ]
    }
);

exit();


docker exec -it shard2-1 mongosh --port 27022

rs.initiate(
    {
      _id : "shard2-1",
      members: [
        { _id : 1, host : "shard2-1:27022" }
      ]
    }
);

exit();


# Инициализация роутера и наполнение его тестовыми данными

docker exec -it mongos_router mongosh --port 27023

sh.addShard( "shard1-1/shard1-1:27021");
sh.addShard( "shard2-1/shard2-1:27022");

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb

for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

db.helloDoc.countDocuments() // будет 1000 документов
exit();


# Проверка на шардах:

docker exec -it shard1-1 mongosh --port 27021

use somedb;
db.helloDoc.countDocuments(); // 492 документа
exit(); 

docker exec -it shard2-1 mongosh --port 27022

use somedb;
db.helloDoc.countDocuments(); // 508 документов
exit(); 
