
# 📦 Cluster MongoDB com Docker (Replica Set - 4 Nós)

Este repositório contém todos os comandos utilizados na configuração de um cluster MongoDB com 4 nós utilizando Docker e Replica Set.

---

## 🧱 Estrutura do Cluster

- `mongo10`: Nó 1 (Primário no início)
- `mongo20`: Nó 2 (Secundário)
- `mongo30`: Nó 3 (Secundário)
- `mongo40`: Nó 4 (Secundário)

Todos estão conectados na mesma `rede Docker` chamada `mongoCluster`.

---

## 🧰 Comandos Utilizados

### 1. Criar rede do Docker
```bash
docker network create mongoCluster
```

### 2. Subir os 4 contêineres MongoDB
```bash
docker run -d --rm --name mongo10 --network mongoCluster -p 27017:27017 mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo10

docker run -d --rm --name mongo20 --network mongoCluster -p 27018:27017 mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo20

docker run -d --rm --name mongo30 --network mongoCluster -p 27019:27017 mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo30

docker run -d --rm --name mongo40 --network mongoCluster -p 27020:27017 mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo40
```

### 3. Acessar o shell do primeiro nó
```bash
docker exec -it mongo10 mongosh
```

### 4. Verifique se o container docker conectado está sendo executado
```bash
docker db.runCommand ({hello:1})
```

### 5. Iniciar o Replica Set
```javascript
rs.initiate(
  {
    _id: "myReplicaSet",
    members: [
      { _id: 0, host: "mongo10:27017" },
      { _id: 1, host: "mongo20:27017" },
      { _id: 2, host: "mongo30:27017" },
      { _id: 3, host: "mongo40:27017" }
    ]
  }
)
```

### 6. Comando exit para sair do container shell
```bash
exit
```

### 7. Verificar o status do Replica Set
```bash
docker exec -it mongo10 mongosh --eval "rs.status()"
```

---

## 📥 Inserção de dados

No shell do MongoDB:
```javascript
use CorporeSystem

db.cliente.insertOne({codigo: 1, nome: "Matheus"});
db.cliente.insertOne({codigo: 2, nome: "Eduardo"});
db.cliente.insertOne({codigo: 3, nome: "Guilherme"});
db.cliente.insertOne({codigo: 4, nome: "Josafá"});
db.cliente.insertOne({codigo: 5, nome: "Davi"});
```

---

## 🔌 Testes de Failover

### 1. Verificar nó primário:
```javascript
rs.isMaster().primary
```

### 2. Parar um nó para simular falha:
```bash
docker stop mongo10
```

### 3. Tentar nova inserção (irá falhar se conectado a um nó secundário):
```javascript
db.cliente.insertOne({codigo: 6, nome: "Rogerio"});
```

### 4. Ver status do novo primário:
```bash
docker exec -it mongo20 mongosh --eval "rs.status()"
```

---

## 🔄 Subir novamente o nó parado
```bash
docker run -d --rm -p 27017:27017 --name mongo10 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo10
```

---

## 📎 Conexão com MongoDB Compass

### String de conexão (exemplo):
```
mongodb://localhost:27017,localhost:27018,localhost:27019,localhost:27020/?replicaSet=myReplicaSet
```

---
