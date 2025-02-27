### **Reindex API Kya Hota Hai?**  

**Reindex API** ka use **Elasticsearch me ek index se doosre index me data copy karne** ke liye hota hai.  

🚀 **Iska Use Kab Hota Hai?**  
1. **Index structure change karne ke liye** (e.g., number of shards badalne ke liye)  
2. **Mapping ya settings update karne ke liye**  
3. **Old index ka data new index me migrate karne ke liye**  
4. **Data filtering aur transformation karne ke liye**  

---

### **🛠 Example: Basic Reindexing**
```http
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  }
}
```
📌 **Isse pura data `old_index` se `new_index` me copy ho jayega.**  

---

### **🎯 Filter Lagake Reindex Karna**
Agar tumhe sirf kuch specific data **filter karke** reindex karna hai, to `query` ka use kar sakte ho:  

```http
POST _reindex
{
  "source": {
    "index": "old_index",
    "query": {
      "range": {
        "price": {
          "gte": 50
        }
      }
    }
  },
  "dest": {
    "index": "new_index"
  }
}
```
📌 **Yeh sirf un documents ko copy karega jisme `price` 50 ya usse zyada hai.**  

---

### **🔄 Data Transform Karna During Reindex**
Agar tumhe **reindexing ke saath data modify bhi karna hai**, to **script** ka use kar sakte ho:  

```http
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  },
  "script": {
    "source": "ctx._source.price = ctx._source.price * 1.1" 
  }
}
```
📌 **Yeh har document ka `price` 10% badha dega jab data `new_index` me jayega.**  

---

### ✅ **Conclusion**
- **Reindex API** ka use **data ko ek index se doosre index me copy/migrate karne** ke liye hota hai.  
- Tum **poora data**, **filtered data**, ya **modified data** reindex kar sakte ho.  
- **Number of shards change karne ke liye** bhi Reindex API ka use hota hai, kyunki **directly shards count change nahi hota**.