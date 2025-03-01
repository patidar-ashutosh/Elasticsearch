# **🔹 `update_by_query` & `delete_by_query` in Elasticsearch**

Elasticsearch me **`update_by_query`** aur **`delete_by_query`** ka use **multiple documents** ko ek sath update ya delete karne ke liye hota hai. Yeh dono operations **search query** ke basis par kaam karte hain.  

> 💡 **Matlab:** Tum **kisi condition ke basis pe ek sath multiple documents ko update ya delete** kar sakte ho bina manually ek-ek document ko update/delete kiye.

---

## **1️⃣ `update_by_query` (Updating Multiple Documents by Query)**  
Iska use **ek sath multiple documents update karne ke liye hota hai** jo kisi particular **condition ko match** karte hain.

### **🔹 Example: Increase Price of All Products in "Electronics" Category**  
Agar **tumhe sabhi "electronics" category ke products ka price 10% se badhana ho**, to tum **`update_by_query`** ka use kar sakte ho:

```http
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.price *= 1.10" 
  },
  "query": {
    "match": {
      "category": "electronics"
    }
  }
}
```
🔹 **Explanation:**  
✔ **`script`**: Yeh batata hai ki kaunsa field update hoga → `ctx._source.price *= 1.10` (price **10% increase ho jayega**)  
✔ **`query`**: Sirf wahi documents update honge jo `"category": "electronics"` wale hain  
✔ **`_update_by_query`** request **batch me process hoti hai**, yani Elasticsearch ek sath multiple documents ko update karega  

---

### **🔹 Example: Add a New Field in Matching Documents**
Agar tum **jitne bhi products `"category": "kitchen"` ke hain, unme ek `"discount": true"` field add karna chahte ho**, to:

```http
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.discount = true"
  },
  "query": {
    "match": {
      "category": "kitchen"
    }
  }
}
```
✔ **Isse sabhi `"category": "kitchen"` wale documents me `"discount": true"` add ho jayega.**  

---

### **🔹 Example: Remove a Field from All Documents**
Agar **tumhe `"out_of_stock"` field ko hata dena ho jitne bhi documents me yeh field present hai**, to:

```http
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.remove('out_of_stock')"
  },
  "query": {
    "exists": {
      "field": "out_of_stock"
    }
  }
}
```
✔ Yeh **sabhi documents me se `"out_of_stock"` field hata dega** jo **is field ko contain karte hain**.  

---

## **2️⃣ `delete_by_query` (Deleting Multiple Documents by Query)**  
Iska use **multiple documents ko ek sath delete karne ke liye** hota hai jo kisi **particular query ko match** karte hain.

### **🔹 Example: Delete All "Out of Stock" Products**  
Agar tumhe **jitne bhi products `"in_stock": 0"` hain unko delete karna ho**, to:

```http
POST /products/_delete_by_query
{
  "query": {
    "term": {
      "in_stock": 0
    }
  }
}
```
✔ **Yeh Elasticsearch me se sabhi `"in_stock": 0"` wale products delete kar dega.**  

---

### **🔹 Example: Delete Old Orders Before 2023**
Agar tumhe **jitne bhi orders `"order_date": "2022-12-31"` ya usse pehle ke hain unko delete karna ho**, to:

```http
POST /orders/_delete_by_query
{
  "query": {
    "range": {
      "order_date": {
        "lte": "2022-12-31"
      }
    }
  }
}
```
✔ **Isse sabhi 2022 ya usse pehle ke orders delete ho jayenge.**  

---

### **🔹 Example: Delete All Documents in an Index**
Agar tumhe **poora index delete nahi karna, sirf uske andar ke sabhi documents delete karne hain**, to:

```http
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```
✔ **Yeh `products` index ke sabhi documents delete karega, lekin index itself delete nahi hoga.**  

---

## **🔹 Key Differences Between `update_by_query` & `delete_by_query`**
| Feature  | `update_by_query` | `delete_by_query` |
|----------|-----------------|-----------------|
| **Function** | Multiple documents ko update karta hai | Multiple documents ko delete karta hai |
| **Uses** | Fields modify karne ke liye | Data cleanup ke liye |
| **Performance** | Thoda slow ho sakta hai kyunki update me scripting lagta hai | Fast hota hai kyunki sirf delete operation perform hota hai |
| **Use Cases** | Price increase, field add/remove, bulk data correction | Expired data delete karna, purani entries remove karna |

---

## **🔹 Conclusion**
✅ **`update_by_query` ka use multiple documents ko update karne ke liye hota hai** jo kisi condition ko match karte hain.  
✅ **`delete_by_query` ka use ek sath multiple documents ko delete karne ke liye hota hai** jo kisi query ko match karte hain.  
✅ **Dono operations search query ke basis pe documents modify/delete karte hain.**  
✅ **Yeh efficient aur fast tarika hai bulk updates aur deletions perform karne ka.**  

💡 **Use these APIs wisely, kyunki yeh irreversible changes perform karte hain!** 🚀

---
[Reference Video Link](https://youtu.be/m9nQM6QH4J4?si=K02zOI1ocx0lT2-T)
---