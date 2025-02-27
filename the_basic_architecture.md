## **📌 Elasticsearch Basic Architecture**  

Elasticsearch ka architecture **distributed** hota hai, jo high availability aur scalability provide karta hai. Is architecture me **cluster, node, index, aur document** key components hote hain.  

---

### **1️⃣ Cluster**  
🔹 **Cluster ek group hota hai jo multiple nodes ka collection hota hai.**  
🔹 **Ek cluster ka ek unique naam hota hai**, jo usko identify karta hai (default: `elasticsearch`).  
🔹 Cluster ke andar multiple **nodes** hote hain jo data ko store karte hain aur queries handle karte hain.  

✅ **Example:**  
```yaml
cluster.name: my-cluster
```

---

### **2️⃣ Node**  
🔹 **Node ek individual server hota hai jo Elasticsearch cluster ka part hota hai.**  
🔹 Ek node **data store kar sakta hai or indexing aur search operations bhi perform kar sakta hai**.  
🔹 Ek cluster me multiple nodes ho sakte hain.  

✅ **Example:**  
```yaml
node.name: my-node
```

---

### **3️⃣ Index**  
🔹 **Index ek logical namespace hota hai jisme similar type ke documents store hote hain.**  
🔹 Har index ka ek unique naam hota hai.  
🔹 Index ke andar **shards** aur **replicas** hote hain jo distributed storage enable karte hain.  

✅ **Example:**  
```bash
PUT /products
```

---

### **4️⃣ Document**  
🔹 **Document Elasticsearch ka smallest unit hota hai jisme JSON format me data store hota hai.**  
🔹 Har document ka ek unique **ID** hota hai jo usko identify karta hai.  
🔹 Documents indexes ke andar store hote hain.  

✅ **Example:**  
```bash
PUT /products/_doc/1
{
  "name": "Laptop",
  "price": 50000
}
```

## Elasticsearch ka data storage flow:

1️⃣ **Data ➝ Document** me store hota hai  
2️⃣ **Document ➝ Index** ke andar store hota hai  
3️⃣ **Index ➝ Node** ke andar store hota hai  
4️⃣ **Node ➝ Cluster** ka part hota hai  

🔹 **Simplified Flow:**  
```
Data → Document → Index → Node → Cluster
```

---

## **📌 Example (Flow in Action)**  
Agar tum ye data store karte ho:  
```bash
PUT /products/_doc/1
{
  "name": "Laptop",
  "price": 50000
}
```
Toh iska flow kuch aisa hoga:  

1️⃣ **Data** → `{ "name": "Laptop", "price": 50000 }`  
2️⃣ **Document** → ID `1` ke saath `products` index me store hoga  
3️⃣ **Index (`products`)** → Elasticsearch ke kisi ek node par store hoga  
4️⃣ **Node** → Ye cluster ka part hoga jo distributed storage manage karega  

---

## **📌 Final Structure**
```
Cluster  
 ├── Node-1  
 │    ├── Index: products  
 │    │    ├── Document ID: 1 → { "name": "Laptop", "price": 50000 }  
 │    │    ├── Document ID: 2 → { "name": "Mobile", "price": 20000 }  
 │    │  
 │    ├── Index: orders  
 │         ├── Document ID: 1 → { "order_id": 101, "amount": 1500 }  
 │  
 ├── Node-2  
 │    ├── Index: products (Replica Shard)
```

---

✅ **Cluster ke andar multiple nodes hote hain.**  
✅ **Node ke andar multiple indexes store ho sakte hain.**  
✅ **Index ke andar multiple documents store hote hain.**  


## **🎯 Summary**  
| **Component** | **Description** |
|-------------|--------------|
| **Cluster** | Multiple nodes ka group |
| **Node** | Cluster ka ek part jo data store karta hai |
| **Index** | Similar documents ka logical group |
| **Document** | JSON format me store hone wala data |