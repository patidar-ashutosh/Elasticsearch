## 📌 **Understanding Document Versioning in Elasticsearch**  

Elasticsearch me **document versioning** ka concept hota hai, lekin ye ek **simple form of versioning** hai. Iska **revision history se koi lena dena nahi hota**, matlab tum **past versions dekh nahi sakte**. Chalo is concept ko **achhe se samajhte hain**!  

---

### **🔹 Elasticsearch Document Versioning Kya Hai?**
Jab bhi tum **Elasticsearch me koi document index karte ho**, to uske saath ek **`_version` field** automatically store hoti hai.  

✔ **Versioning ka basic rule:**  
- **Pehli baar document create hota hai to `_version = 1` hota hai.**  
- **Har update pe `_version` increment hota hai (`+1`).**  
- **Agar document delete hota hai, to `_version` number ko 60 seconds tak retain kiya jata hai.**  

📌 **Ek important baat:**  
👉 Ye versioning **document ke changes ka history maintain nahi karti**.  
👉 **Sirf latest version store hota hai** (Tum **purane versions wapas nahi la sakte**).  

---

### **🔹 Example: Internal Versioning**  
```http
# Pehla document create karte hain
PUT /products/_doc/1
{
  "name": "Laptop",
  "brand": "Dell",
  "price": 75000
}
```
🔹 **Response:**  
```json
{
  "_index": "products",
  "_id": "1",
  "_version": 1,
  "result": "created"
}
```
➡ `_version: 1` indicate karta hai ki ye **pehli baar index hua hai**.  

---

### **🔹 Version Increment on Update**  
```http
# Update the document
POST /products/_update/1
{
  "doc": {
    "price": 72000
  }
}
```
🔹 **Response:**  
```json
{
  "_index": "products",
  "_id": "1",
  "_version": 2,
  "result": "updated"
}
```
➡ **Update hone ke baad `_version: 2` ho gaya!**  

---

### **🔹 Delete aur Re-index ke Baad Version Handling**
```http
# Delete the document
DELETE /products/_doc/1
```
➡ **Ab ye document delete ho gaya, lekin Elasticsearch 60 seconds tak iska `_version` yaad rakhega.**  

```http
# Same ID ka naya document create karna
PUT /products/_doc/1
{
  "name": "Gaming Laptop",
  "brand": "Asus",
  "price": 90000
}
```
Agar ye **60 sec ke andar hua to `_version = 3`** hoga,  
Agar **60 sec ke baad kiya to `_version = 1` se reset hoga.**  

---

## **🔹 Internal vs External Versioning**
Elasticsearch me **2 tarah ki versioning hoti hai**:  

| **Type**       | **Kaise Work Karta Hai?** |
|---------------|--------------------------|
| **Internal**  | Elasticsearch automatically `_version` track karta hai. (Default behavior) |
| **External**  | Tum apni **database ya kisi aur system** me version track karke Elasticsearch ko bata sakte ho. |

### **Example: External Versioning**
```http
PUT /products/_doc/1?version=5&version_type=external
{
  "name": "Smartphone",
  "brand": "Samsung",
  "price": 50000
}
```
➡ **Yaha `_version = 5` explicitly diya gaya hai**.  
➡ **Yeh tabhi accept hoga agar jo current `_version` hai wo 5 se chhota ho.**  

---

## **🔹 Why Does Elasticsearch Use Versioning?**
✔ **Optimistic Locking (Concurrency Control):** Prevents overwriting of data in multi-user environments.  
✔ **Consistency Maintain Karna:** Ensures updates happen sequentially.  
✔ **Failure Recovery:** Version helps in ensuring correct data replication.  

---

## **🔹 Conclusion**
1️⃣ **Elasticsearch sirf latest version store karta hai.**  
2️⃣ **_version field automatically maintain hoti hai.**  
3️⃣ **Document update hone pe version badhta hai.**  
4️⃣ **Agar document delete ho jaye, to bhi version 60 sec tak retain hota hai.**  
5️⃣ **External versioning me tum khud version define kar sakte ho.**  

👉 **Ab `_version` field ka major use nahi hota**, kyunki **primary terms aur sequence numbers ne isko replace kar diya hai**.  
