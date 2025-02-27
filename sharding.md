## **📌 Sharding in Elasticsearch**  

**Sharding** ek technique hai jo **Elasticsearch** me **large data sets ko efficiently store aur search** karne ke liye use hoti hai. Jab **ek single index ka data bahut bada ho jaye**, to **Elasticsearch usse multiple parts me tod deta hai**, aur un single-single part ko **"shard"** kehte hai.  

### **🎯 Why Sharding? (Sharding Ki Zaroorat Kyun Hoti Hai?)**  
1️⃣ **Scalability** → Bada data ek server me store nahi ho sakta, isliye shards alag-alag nodes pe store hote hain.  
2️⃣ **Performance** → Multiple shards hone se **parallel processing** hoti hai, jisse search fast ho jata hai.  
3️⃣ **Fault Tolerance** → Agar ek shard fail ho jaye, to replica shards se uska backup le sakte hain.  

## Example
**ek index ka size 600GB hai**, aur tumhare paas **do nodes (A & B) hain, jinka individual storage 500GB** hai, to **600GB ka pura index ek node me fit nahi hoga**.  

### **🚨 Problem: Index ka size ek node se bada hai!**  
- **Node A & Node B** dono milkar **1000GB (1TB) storage** provide kar rahe hain.  
- **Index ka size 600GB hai**, jo **ek single node me store nahi ho sakta** kyunki ek node ki max storage **500GB** hai.  
- **Agar index sirf ek shard me bana**, to wo ek node me hi store hoga, lekin **shard ki max limit ek node ki limit se zyada hai**.  

➡️ **Solution:** Index ko multiple shards me todna hoga!  

## **📌 Solution: Sharding Ka Use Karke Index Ko Nodes Me Distribute Karna**  
✅ **Agar tum index ko 3 shards me divide kar do**, to har shard alag node pe store ho sakta hai.  
✅ **600GB ka data agar 3 shards me split ho to:**  
- **Shard 1 → 200GB**  
- **Shard 2 → 200GB**  
- **Shard 3 → 200GB**  

✅ **Agar Elasticsearch ko auto-allocate karne do, to:**  
| **Node** | **Shard** | **Size** |
|---------|---------|---------|
| **Node A** | Shard 1 | 200GB |
| **Node A** | Shard 2 | 200GB |
| **Node B** | Shard 3 | 200GB |
| **Total Usage** | | **600GB** (Distributed across nodes) |

🚀 **Ab pura index store ho sakta hai, kyunki shards alag-alag nodes pe distribute ho gaye!**  

---

## **📌 Sharding Ka Working Mechanism**  
✅ Jab tum **ek index create karte ho**, to **Elasticsearch us index ko multiple shards me divide karta hai**.  
✅ **Default me har index ke 1 primary shard aur 1 replica shard hota hai** (Par tum isko customize bhi kar sakte ho).  
✅ **Har shard ek chote index ki tarah kaam karta hai**, jo apne data ko alag store karta hai.  

📌 **Example**:  
Maan lo tumne **"products"** naam ka ek index banaya, jisme **10 million documents** store hone hain. Agar **Elasticsearch ek hi shard me saara data store kare, to ek node overload ho sakti hai.**  

### **➡️ Solution: Index Ko 3 Shards Me Divide Karo**  
```plaintext
Index: products
Shards: 
  - Shard 1 → (3 million documents)
  - Shard 2 → (3 million documents)
  - Shard 3 → (4 million documents)
```
✅ Ab jab tum search karoge, to Elasticsearch **parallelly** teenon shards pe query run karega, jo **fast response** dega!  

---

## **📌 Example: Shards Configuration in Elasticsearch**  
**Index create karte waqt tum primary aur replica shards define kar sakte ho:**  

```bash
PUT /products
{
  "settings": {
    "number_of_shards": 3,    # Primary Shards
    "number_of_replicas": 1   # Replica Shards
  }
}
```
✅ **Iska Matlab:**  
- **3 primary shards** honge (data divide hone ke liye).  
- **Har shard ka ek replica hoga** (backup ke liye).  
- **Total 6 shards honge** (3 primary + 3 replicas).  

---

## **📌 Real-Life Example: Library Management System**  
Socho tumhari **library me 1 crore books ka data store karna hai**. Agar tum saari books ka data **ek hi database me store kar doge, to system slow ho jayega**.  

➡️ **Solution:**  
Tum **books ka data alag-alag categories (shards) me divide kar do**:  

| **Shard** | **Category**  | **No. of Books** |
|---------|------------|---------------|
| **Shard 1** | Science Books  | 30 Lakh  |
| **Shard 2** | Fiction Books  | 40 Lakh  |
| **Shard 3** | History Books  | 30 Lakh  |

✅ Ab jab koi student **science books search karega, to sirf Shard 1 me search hoga, pura database check nahi hoga** → **System fast ho jayega!** 🚀  

---

## **🎯 Summary**
✅ **Sharding data ko multiple parts me todne ki technique hai**, jo **Elasticsearch me performance aur scalability improve karti hai**.  
✅ **Har shard ek independent chhote index ki tarah hota hai**, jo apna data store karta hai.  
✅ **Shards Elasticsearch ke multiple nodes me distribute hote hain**, jo system ko **fault-tolerant** aur **fast** banata hai.  
✅ **Sharding ka real-life example library management system hai**, jisme alag-alag shards alag-alag categories ko store karte hain.  