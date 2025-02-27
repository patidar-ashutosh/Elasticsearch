## 🚀 How Elasticsearch Writes Data (Step by Step)  

Elasticsearch me data write hone ka process kuch steps me hota hai. Chalo ek **real-world example** ke saath samajhte hain.  

---

### **Example Scenario:**  
Maan lo ek **e-commerce app** hai jisme **products** ka data store ho raha hai.  
Hum ek **naya product "Laptop"** Elasticsearch me store karenge.  

📌 **Index Name:** `products`  
📌 **Document ID:** `101`  
📌 **Data:**  
```json
{
    "name": "Laptop",
    "brand": "Dell",
    "price": 75000,
    "stock": 20
}
```

---

### **🛠 Steps: How Data is Written in Elasticsearch**

🔹 **1. Client Request**  
   - Hum ek **HTTP request** bhejte hain Elasticsearch ko.  
   - Example:
     ```http
     PUT /products/_doc/101
     {
         "name": "Laptop",
         "brand": "Dell",
         "price": 75000,
         "stock": 20
     }
     ```
   - **Elasticsearch ka "coordinating node" request receive karta hai.**  

---

🔹 **2. Routing: Decide Shard**  
   - **Elasticsearch routing formula** lagata hai aur decide karta hai ki **document kaunsa shard me jayega**.  
   - Default formula:  
     ```
     shard = hash(document_id) % number_of_primary_shards
     ```
   - **Maan lo cluster me 3 primary shards hai (`shard-0`, `shard-1`, `shard-2`).**  
   - Suppose `101` ka **hashing calculate karke `shard-1` select hota hai**.  

---

🔹 **3. Primary Shard me Data Write Hota Hai**  
   - Ab **coordinating node request ko `shard-1` ke primary shard par bhejta hai**.  
   - Primary shard **document ko store karta hai** aur **ACK (Acknowledgment) bhejta hai**.  

---

🔹 **4. Replica Shards me Copy Hota Hai**  
   - Agar **replica shards configured hain** to Elasticsearch **automatically same document replica shards me copy karta hai**.  
   - Suppose **shard-1 ka replica `shard-1 (replica)` node B me hai**, to waha bhi copy ho jayega.  

---

🔹 **5. Success Response**  
   - Jaise hi **primary shard aur replica shards me data store ho jata hai**, Elasticsearch **client ko success response bhejta hai**:  
   ```json
   {
       "_index": "products",
       "_id": "101",
       "result": "created",
       "_shards": {
           "total": 2,
           "successful": 2,
           "failed": 0
       }
   }
   ```
   - **Iska matlab document successfully store ho gaya! 🎉**  

---

### **📌 Summary (Flow Chart)**
1️⃣ Client request bhejta hai (PUT request)  
2️⃣ **Coordinating node request accept karta hai**  
3️⃣ **Routing formula decide karta hai ki kaunsa primary shard use hoga**  
4️⃣ **Primary shard me document store hota hai**  
5️⃣ **Replica shards me data copy hota hai**  
6️⃣ **Success response client ko bhej diya jata hai**  

---

### **📝 Example Query to Verify Data**
Data store ho gaya ya nahi check karne ke liye:  
```http
GET /products/_doc/101
```
Response:
```json
{
    "_index": "products",
    "_id": "101",
    "_version": 1,
    "_source": {
        "name": "Laptop",
        "brand": "Dell",
        "price": 75000,
        "stock": 20
    }
}
```

---

### **🚀 Conclusion**
👉 **Write process me pehle routing hoti hai, fir primary shard me data store hota hai, aur phir replica me copy hota hai.**  
👉 **Agar koi shard fail ho jaye to replica se recover ho sakta hai.**  
👉 **Data consistency aur availability Elasticsearch ke distributed architecture ki wajah se maintain hoti hai.**  

---
[Reference Video Link](https://youtu.be/Zei4mlPatpY?si=z6ZTi3dpVBuuKi3K)
---