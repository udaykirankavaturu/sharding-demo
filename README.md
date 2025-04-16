# 🧩 MongoDB Sharding Demo with Docker

This project demonstrates how to manually set up and explore **MongoDB sharding** using Docker, step by step.

---

## 📦 Prerequisites

- [Docker](https://www.docker.com/products/docker-desktop)
- `mongosh` shell (or use inside containers)

---

## 🛠️ Architecture Overview

This setup includes:

- 🗃️ 2 Shards (each a replica set)
- ⚙️ 1 Config Server Replica Set
- 📡 1 Mongos Router

---

## 🚀 Step-by-Step Setup

### Step 1: Clone and Start Containers

```bash
git clone https://github.com/your-username/mongodb-sharding-demo.git
cd mongodb-sharding-demo
```

```
docker compose up -d
```

This will launch:

    configsvr replica set

    shard1 replica set

    shard2 replica set

    mongos router

### 🔁 Step 2: Initialize Replica Sets

1️⃣ Init Config Server

```
docker exec -it configsvr mongosh
```

Inside shell

```
rs.initiate({
_id: "configReplSet",
configsvr: true,
members: [{ _id: 0, host: "configsvr:27017" }]
})
```

```
exit
```

2️⃣ Init Shard 1

```
docker exec -it shard1 mongosh --port 27018
```

Inside shell

```
rs.initiate({
_id: "shard1ReplSet",
members: [{ _id: 0, host: "shard1:27018" }]
})
```

```
exit
```

3️⃣ Init Shard 2

```
docker exec -it shard2 mongosh --port 27019
```

Inside shell

```
rs.initiate({
_id: "shard2ReplSet",
members: [{ _id: 0, host: "shard2:27019" }]
})
```

```
exit
```

### 🌐 Step 3: Connect to Mongos Router

```
docker exec -it mongos mongosh --port 27020
```

### Step 4: Add Shards to Cluster

```
sh.addShard("shard1ReplSet/shard1:27018")
sh.addShard("shard2ReplSet/shard2:27019")
```

Confirm with:

```
sh.status()
```

### 💾 Step 5: Enable Sharding on Database + Collection

```
sh.enableSharding("shardingDemo")
```

```
db.users.createIndex({ userId: 1 })
sh.shardCollection("shardingDemo.users", { userId: 1 })
use shardingDemo
```

### 📥 Step 6: Insert Data

Small Set

```
db.users.insertMany([
{ userId: 1, name: "Alice", region: "US" },
{ userId: 2, name: "Bob", region: "EU" },
{ userId: 3, name: "Charlie", region: "APAC" }
])
```

Large Dataset (to trigger chunk splits)

```
for (let i = 4; i <= 10000; i++) {
db.users.insertOne({ userId: i, name: "User" + i, region: "Region" + (i % 5) });
}
```

### 📊 Step 7: Monitor Shard Distribution

```
db.users.getShardDistribution()
```

### 🚚 (optional)Step 8: Move Chunks Between Shards

```
sh.splitAt("shardingDemo.users", { userId: 5000 })
sh.moveChunk("shardingDemo.users", { userId: 5000 }, "shard2ReplSet")
```

Recheck with:

db.users.getShardDistribution()
sh.status()

### 🧹 Cleanup

```
docker compose down -v
```

## 🧠 Notes

MongoDB doesn't split data into chunks until ~64MB is reached (or manually split).

The balancer will auto-redistribute chunks if autosplit and balancing are enabled (check with sh.status()).

Manual splits and moves help for demo purposes.
