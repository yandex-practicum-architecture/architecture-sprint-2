1. Запуск приложения:

.\mongo-sharding-repl>docker compose build

.\mongo-sharding-repl>docker compose up -d

2. Настройка:

.\mongo-sharding-repl>docker compose exec -T router01 mongosh

Вывод:
		Current Mongosh Log ID: 67fd3e2415215a3c276b140a
		Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
		Using MongoDB:          8.0.6
		Using Mongosh:          2.4.2

		For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

		------
		   The server generated these startup warnings when booting
		   2025-04-14T16:48:44.997+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
		   2025-04-14T16:48:44.997+00:00: You are running this process as the root user, which is not recommended
		------

3. Дополнительно можно проверить (описано в compose.yaml 
и настраивается в скриптах запуска , 
см. каталог scripts):

[direct: mongos] test> sh.addShard("rs-shard-01/shard-01-node-a:27017");
		MongoServerError[IllegalOperation]: A shard named rs-shard-01 containing the replica set 'rs-shard-01' already exists
		
[direct: mongos] test> sh.addShard("rs-shard-02/shard-02-node-a:27017");
		MongoServerError[IllegalOperation]: A shard named rs-shard-02 containing the replica set 'rs-shard-02' already exists

4. Включаем шардирование:

[direct: mongos] test> sh.enableSharding("somedb");

Вывод:
		{
		  ok: 1,
		  '$clusterTime': {
			clusterTime: Timestamp({ t: 1744649792, i: 8 }),
			signature: {
			  hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
			  keyId: Long('0')
			}
		  },
		  operationTime: Timestamp({ t: 1744649792, i: 5 })
		}

5. Настраиваем шардирование коллекции:

[direct: mongos] test> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" });

Вывод:
		{
		  collectionsharded: 'somedb.helloDoc',
		  ok: 1,
		  '$clusterTime': {
			clusterTime: Timestamp({ t: 1744649799, i: 47 }),
			signature: {
			  hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
			  keyId: Long('0')
			}
		  },
		  operationTime: Timestamp({ t: 1744649799, i: 46 })
		}

6. Перключаемся на бд somedb и добавляем данные:

[direct: mongos] test> use somedb;

Вывод:
		switched to db somedb

[direct: mongos] somedb> for(var i = 0; i < 1000; i++) db.helloDoc.insertOne({age:i, name:"ly"+i})

Вывод:
		{
		  acknowledged: true,
		  insertedId: ObjectId('67fd3e8315215a3c276b17f2')
		}

7. Завершаем сессию работы с бд:

[direct: mongos] somedb> exit

8. Заходим в консоль первого шарда для проверки состояния:

.\mongo-sharding-repl>docker exec -it shard-01-node-a mongosh --port 27017

Вывод:
		Current Mongosh Log ID: 67fd3e98b9099f5f886b140a
		Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
		Using MongoDB:          8.0.6
		Using Mongosh:          2.4.2

		For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/


		To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
		You can opt-out by running the disableTelemetry() command.

		------
		   The server generated these startup warnings when booting
		   2025-04-14T16:48:10.150+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
		   2025-04-14T16:48:22.301+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
		   2025-04-14T16:48:22.301+00:00: You are running this process as the root user, which is not recommended
		   2025-04-14T16:48:22.301+00:00: For customers running the current memory allocator, we suggest changing the contents of the following sysfsFile
		   2025-04-14T16:48:22.301+00:00: We suggest setting the contents of sysfsFile to 0.
		   2025-04-14T16:48:22.301+00:00: vm.max_map_count is too low
		   2025-04-14T16:48:22.301+00:00: We suggest setting swappiness to 0 or 1, as swapping can cause performance problems.
		------

9. Переключаемся на соответствующую бд и проверяем количество элементов в шарде:

rs-shard-01 [direct: primary] test> use somedb;

Вывод:
		switched to db somedb

rs-shard-01 [direct: primary] somedb> db.helloDoc.countDocuments();

Вывод:
		492

10. Завершаем сессию работы с бд:

rs-shard-01 [direct: primary] somedb> exit

11. Аналогично для второго шарда, заходим в консоль:

.\mongo-sharding-repl>docker exec -it shard-02-node-a mongosh --port 27017

Вывод:
		Current Mongosh Log ID: 67fd3eb47c71d3f1ea6b140a
		Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
		Using MongoDB:          8.0.6
		Using Mongosh:          2.4.2

		For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/


		To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
		You can opt-out by running the disableTelemetry() command.

		------
		   The server generated these startup warnings when booting
		   2025-04-14T16:48:09.458+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
		   2025-04-14T16:48:19.255+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
		   2025-04-14T16:48:19.256+00:00: You are running this process as the root user, which is not recommended
		   2025-04-14T16:48:19.256+00:00: For customers running the current memory allocator, we suggest changing the contents of the following sysfsFile
		   2025-04-14T16:48:19.256+00:00: We suggest setting the contents of sysfsFile to 0.
		   2025-04-14T16:48:19.256+00:00: vm.max_map_count is too low
		   2025-04-14T16:48:19.256+00:00: We suggest setting swappiness to 0 or 1, as swapping can cause performance problems.
		------

12. Переключаемся на соответствующую бд и проверяем количество элементов в шарде:

rs-shard-02 [direct: secondary] test> use somedb;

Вывод:
		switched to db somedb

rs-shard-02 [direct: secondary] somedb> db.helloDoc.countDocuments();

Вывод:
		508

13. Завершаем сессию работы с бд:
rs-shard-02 [direct: secondary] somedb> exit

14. Проверяем работу endpoints, конфигурация:

GET http://localhost:8080/

Вывод:
		{
		  "mongo_topology_type": "Sharded",
		  "mongo_replicaset_name": null,
		  "mongo_db": "somedb",
		  "read_preference": "Primary()",
		  "mongo_nodes": [
			[
			  "router01",
			  27017]
		  ],
		  "mongo_primary_host": null,
		  "mongo_secondary_hosts": [],
		  "mongo_address": [
			"router01",
			27017],
		  "mongo_is_primary": true,
		  "mongo_is_mongos": true,
		  "collections": {
			"helloDoc": {
			  "documents_count": 1000
			}
		  },
		  "shards": {
			"rs-shard-01": "rs-shard-01/shard01-a:27017,shard01-b:27017,shard01-c:27017",
			"rs-shard-02": "rs-shard-02/shard02-a:27017,shard02-b:27017,shard02-c:27017"
		  },
		  "cache_enabled": false,
		  "status": "OK"
		}

15. Swagger API:
	GET http://localhost:8080/docs

16. Коллекция пользователей, endpoint:
GET http://localhost:8080/helloDoc/users

Вывод:
		{
		  "users": [
			{
			  "id": "67fd3e7915215a3c276b140b",
			  "age": 0,
			  "name": "ly0"
			},
			{
			  "id": "67fd3e7915215a3c276b140c",
			  "age": 1,
			  "name": "ly1"
			},
			{
			  "id": "67fd3e7915215a3c276b140d",
			  "age": 2,
			  "name": "ly2"
			},
		...

			{
			  "id": "67fd3e8315215a3c276b17f0",
			  "age": 997,
			  "name": "ly997"
			},
			{
			  "id": "67fd3e8315215a3c276b17f1",
			  "age": 998,
			  "name": "ly998"
			},
			{
			  "id": "67fd3e8315215a3c276b17f2",
			  "age": 999,
			  "name": "ly999"
			}
		  ]
		}

17. Количество эелментов в коллекции пользователей, endpoint:

GET http://localhost:8080/helloDoc/count

Вывод:
		{
		  "status": "OK",
		  "mongo_db": "somedb",
		  "items_count": 1000
		}

18. Завершаем работу приложения:

.\mongo-sharding-repl>docker compose down
