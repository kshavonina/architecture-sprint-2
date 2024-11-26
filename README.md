# sharding-repl-cache

Дизайн решения представлен в файле task1_v5.drawio в корне проекта.
После всех шагов, приведенных в данной инструкции, приложение будет доступно по адресу http://localhost:8080.

## Запускаем compose.yaml

docker compose up -d


## Инициализация сервера конфигурации

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

## Инициализация шардов с репликами

docker exec -it shard1-1 mongosh --port 27021

rs.initiate(
    {
      _id : "rs0",
      members: [
        { _id : 0, host : "shard1-1:27021" },
        { _id : 1, host : "shard1-2:27024" },
        { _id : 2, host : "shard1-3:27025" }
      ]
    }
);

exit();


docker exec -it shard2-1 mongosh --port 27022

rs.initiate(
    {
      _id : "rs1",
      members: [
        { _id : 3, host : "shard2-1:27022" },
        { _id : 4, host : "shard2-2:27026" },
        { _id : 5, host : "shard2-3:27027" }
      ]
    }
);

exit();


## Инициализация роутера и наполнение его тестовыми данными

docker exec -it mongos_router mongosh --port 27023

sh.addShard( "rs0/shard1-1:27021");
sh.addShard( "rs1/shard2-1:27022");

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb

for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

db.helloDoc.countDocuments() // будет 1000 документов
exit();


## Проверка на шардах (опционально):

docker exec -it shard1-1 mongosh --port 27021

use somedb;
db.helloDoc.countDocuments(); // 492 документа
exit(); 

docker exec -it shard1-3 mongosh --port 27025

use somedb;
db.helloDoc.countDocuments(); // тоже 492 документа
exit(); 

docker exec -it shard2-1 mongosh --port 27022

use somedb;
db.helloDoc.countDocuments(); // 508 документов
exit(); 
