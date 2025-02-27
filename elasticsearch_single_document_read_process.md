### **📌 Elasticsearch Single Document Read Process**  
Jab bhi tum **ek single document fetch** karne ke liye `GET /index/_doc/{id}` request bhejte ho, to **Elasticsearch internally multiple steps follow karta hai**. Chalo **step-by-step process** samajhte hain! 🚀  

---

### **🔹 Step 1: Coordinating Node Pe Request Aati Hai**
Jab tum **GET request** Elasticsearch cluster ko bhejte ho:
```plaintext
GET /products/_doc/1
```
✅ Ye request **coordinating node** ke paas jaati hai (jo bhi node request receive karti hai).  
✅ Coordinating node **iska shard locate karti hai**, taaki data efficiently fetch ho sake.  

---

### **🔹 Step 2: Routing Formula Se Shard Identify Hota Hai**
Elasticsearch **document ID ka hash function** nikalta hai aur routing formula lagata hai:
```plaintext
shard = hash(document_id) % number_of_primary_shards
```
✅ Maan lo **index me 5 primary shards hain**, to:
```plaintext
shard = hash(1) % 5 = 3
```
Matlab **ye document `shard-3` me pada hoga**.

---

### **🔹 Step 3: Primary Ya Replica Se Data Fetch Hota Hai (Adaptive Replica Selection - ARS)**
✅ Elasticsearch me **data ya to primary shard pe hoga ya replica shard pe**.  
✅ **ARS (Adaptive Replica Selection)** decide karta hai ki **fastest node** kaunsi hai aur wahin se data fetch hota hai.  

Maan lo **`shard-3` primary `Node A` pe hai** aur **iska replica `Node B` pe hai**.  
✔ Agar `Node A` free hai, to **wahan se data fetch hoga**.  
✔ Agar `Node A` busy hai, to **replica `Node B` se data fetch hoga**.  

---

### **🔹 Step 4: Coordinating Node Response Wapas Bhejta Hai**
✅ Data fetch hone ke baad **coordinating node client ko response wapas bhejti hai**:  
```json
{
    "_index": "products",
    "_id": "1",
    "_version": 3,
    "found": true,
    "_source": {
        "name": "MacBook Pro"
    }
}
```
✅ Ye response **actual stored document** return karta hai jo **_source** field me hota hai.  

---

### **🎯 Final Flow Summary**
1️⃣ **Client Request:** `GET /products/_doc/1`  
2️⃣ **Coordinating Node** request receive karta hai.  
3️⃣ **Routing Formula** `shard = hash(1) % 5 = 3` shard select karta hai.  
4️⃣ **ARS (Adaptive Replica Selection)** fastest node choose karta hai (Primary ya Replica).  
5️⃣ **Shard se Document Read Hota Hai.**  
6️⃣ **Response Client ko Wapas Milta Hai.** 🚀  

---

### **🔥 Bonus: Diagrammatic Representation**
```
Client ---> Coordinating Node ---> Finds Shard Using Routing ---> ARS Selects Fastest Shard (Primary/Replica) ---> Fetches Data ---> Returns to Client
```

---

### **✨ Important Points**
✅ **GET request sirf 1 shard pe jati hai, saare shards pe nahi.**  
✅ **Adaptive Replica Selection (ARS)** optimize karta hai ki kaha se data milega.  
✅ **Single document fetch bahut fast hota hai**, kyunki sirf **ek hi shard** involve hota hai.  

---

### **🚀 Conclusion**
✔ Jab tum **`GET /index/_doc/id`** request bhejte ho, to **Coordinating Node** request leta hai.  
✔ **Routing formula** ke basis pe **correct shard select hota hai**.  
✔ **Adaptive Replica Selection (ARS)** sabse fast shard (Primary ya Replica) choose karta hai.  
✔ **Data fetch hota hai aur response client ko milta hai!** 🚀  

---
[Reference Video Link](https://youtu.be/oz9PecSxE74?si=rQkFyvgPNj5r8a-9)
---