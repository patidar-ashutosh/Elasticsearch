## **🔄 Routing in Elasticsearch**  

### **📌 Routing Kya Hai?**  
Elasticsearch me **routing** ka use hota hai data ko **specific shards** me store aur retrieve karne ke liye. Default behavior me Elasticsearch **document ID ke hash** ko use karke decide karta hai ki document kis shard me jayega.  

👉 **Par agar hume apna custom routing set karna ho to hum `routing` parameter ka use kar sakte hain.**  

---

## **🚀 Kaise Work Karta Hai? (Example)**  

Maan lo hamare paas **products** ka index hai jisme alag-alag categories ke products store hain.  

### **1️⃣ Default Routing (Auto)**
Agar hum **custom routing nahi dete**, to Elasticsearch **document ID ka hash** calculate karke **shard decide** karega.  

```json
POST /products/_doc/1
{
    "name": "iPhone 15",
    "category": "mobile"
}
```
🔹 **Yaha Elasticsearch apne hisaab se shard decide karega.**  

---

### **2️⃣ Custom Routing (Manual)**
Agar hum chahte hain ki **same category wale products ek hi shard me jaye**, to hum **routing parameter** specify kar sakte hain:  

```json
POST /products/_doc/2?routing=mobile
{
    "name": "Samsung Galaxy S23",
    "category": "mobile"
}
```
🔹 **Ab saare `mobile` category wale products ek hi shard me store honge.**  
🔹 **Yadi `category = "laptop"` hoti to wo alag shard me jata.**  

---

### **3️⃣ Query with Routing**
Agar hum koi **specific routed shard** se data fetch karna chahte hain to routing parameter ke sath query kar sakte hain:  

```json
GET /products/_doc/2?routing=mobile
```
🔹 **Ye query sirf `mobile` category wale shard pe jayegi, pura cluster scan nahi karegi.**  
🔹 **Faster Search 🚀 kyunki unnecessary shards query nahi hote.**  

---

## **🎯 Conclusion**
✅ **Routing performance optimize karne ke liye use hoti hai.**  
✅ **Same category ya related data ko ek hi shard me store karne ke liye helpful hai.**  
✅ **Query performance improve hoti hai kyunki unnecessary shards ko scan nahi karna padta.**  

🔥 **Use case:** Multi-tenant applications jisme har customer ka data apne specific shard me store hota hai. 🚀

---
---
---


## **✅ Routing Working With Store And Read**

📌 **Jab document store hota hai:**  
1. Routing decide karta hai ki **kaunsa primary shard use hoga**.  
2. Document **usi primary shard me store hota hai**.  
3. Phir **replica shards me copy hoti hai** (agar configured hai).  

📌 **Jab document read hota hai:**  
- Routing **primary ya replica dono me se kisi ek shard se data fetch kar sakta hai** for better performance.  

🚀 **Conclusion:**  
👉 **Write operations (store/update/delete) me sirf primary shard use hota hai.**  
👉 **Read operations (search/get) me primary ya replica shard koi bhi use ho sakta hai.**  
👉 **Routing formula hamesha same rehta hai, chahe write ho ya read.**  

---
---
---


### **📌 Routing Ka Formula Kya Hai?**  
Elasticsearch **default routing** ke liye ek **formula** use karta hai jo **document ID ka hash nikal kar usse shard count se mod** karta hai.  

### **📌 Default Routing Formula**  
```plaintext
shard = hash(document_id) % number_of_primary_shards
```
🔹 **document_id** → Jo bhi hum document insert kar rahe hain uska ID.  
🔹 **hash(document_id)** → Elasticsearch internally ek hash function use karta hai jo ID ka hash generate karta hai.  
🔹 **number_of_primary_shards** → Index ke total **primary shards** ka count.  

**📌 Example:**  
Maan lo ek index hai **products** jisme **5 primary shards** hain.  

Agar hum ek document insert karte hain:  
```json
POST /products/_doc/1
{
    "name": "iPhone 15"
}
```
To Elasticsearch ye calculate karega:  
```plaintext
shard = hash(1) % 5
```
Agar `hash(1) = 123456789` aaya to:  
```plaintext
shard = 123456789 % 5 = 4
```
To ye document **shard-4** me store ho jayega.  

---

### **📌 Custom Routing Formula**  
Agar hum **custom routing** specify karte hain to formula thoda change ho jata hai:  
```plaintext
shard = hash(custom_routing_key) % number_of_primary_shards
```
Example:  
```json
POST /products/_doc/2?routing=mobile
{
    "name": "Samsung Galaxy S23",
    "category": "mobile"
}
```
🔹 **Yaha `mobile` ka hash nikala jayega** aur mod operation ke through shard decide hoga.  
🔹 **Same routing key (`mobile`) dene se same shard milega, jo query performance improve karega.**  

---

## **🎯 Conclusion**
✅ **Default routing document ID ka hash use karti hai.**  
✅ **Custom routing me hum khud decide kar sakte hain kaunsa shard use hoga.**  
✅ **Same routing key use karne se related data ek hi shard me store hota hai, jo performance optimize karta hai.** 🚀

---
---
---

### **📌 Kyu Hum Index Ke Number of Shards Ko Change Nahi Kar Sakte?**  
✅ Elasticsearch me **shard selection formula** depend karta hai **total primary shards** pe:  
```plaintext
shard = hash(document_id) % number_of_primary_shards
```
✅ Agar hum **index ke primary shards ko badal dete hain**, to **modulus (`%`) ka result change ho jata hai**, jiska matlab **documents naye shards me map nahi honge** aur **search karne pe milenge hi nahi**!  

---

### **📌 Example:**
Maan lo humne ek index banaya jisme **5 primary shards** hain:
```json
PUT /products
{
    "settings": {
        "number_of_shards": 5,
        "number_of_replicas": 1
    }
}
```
Aur hum ek document insert karte hain:
```json
POST /products/_doc/1
{
    "name": "MacBook Pro"
}
```
#### **📌 Shard Selection Calculation**
```plaintext
shard = hash(1) % 5
shard = 123456789 % 5 = 4
```
To **ye document `shard-4` me jayega**.  

---

### **📌 Kya Hoga Agar Hum Shards Change Kar Dein?**  
Agar hum **number_of_shards = 6** kar dete hain, to **formula change ho jayega**:
```plaintext
shard = hash(1) % 6
shard = 123456789 % 6 = 3  ❌
```
Matlab **document ab shard-3 me expect hoga**, par **actual me wo shard-4 me pada hoga**, to **Elasticsearch ko wo milega hi nahi!** 😱  

---

### **📌 Solution:**
🔹 **Primary shards ko create hone ke baad change nahi kar sakte.**  
🔹 Agar shards badalne ka zaroori ho to **Reindex API** ka use karke naye index me data migrate karna padega. [Learn More About Reindex API](./reindex_api.md)  
🔹 **Number of replicas** badal sakte ho, kyuki wo data ko affect nahi karta.  

---

### **🎯 Conclusion**
✅ **Shards ko badalne se routing formula fail ho sakta hai.**  
✅ **Documents physically exist karte hain, par Elasticsearch ko nahi milte.**  
✅ **Agar shards badalne hi hain to naye index me reindex karna padega.** 🚀



---
[Reference Video Link](https://youtu.be/hP6i_lh3A60?si=y0jxR52rmwPBkJZy) 
---