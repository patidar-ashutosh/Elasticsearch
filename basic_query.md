
# Elasticsearch Index and Document Operations

## 🗑 Delete an Index
```http
DELETE /indexName
```

## 🆕 Create a New Index with Configuration Settings
```http
PUT /products
{
    "settings": {
        "number_of_shards": 2,
        "number_of_replicas": 2
    }
}
```

## 📄 Create a Document
```http
POST /products/_doc
{
    "name" : "Coffee Maker",
    "price" : 65,
    "in_stock" : 33    
}
```

## 📄 Create a Document with a Specific ID
```http
PUT /products/_doc/100
{
    "name" : "Toaster",
    "price" : 46,
    "in_stock" : 3    
}
```

## 🔍 Retrieve a Document
```http
GET /products/_doc/100
```
- If `"found": true` → Document exists, and `_source` contains data.
- If `"found": false` → Document does not exist, and `_source` is absent.

## ✏ Update a Document
```http
POST /products/_update/100
{
    "doc": {
        "in_stock" : 2
    }
}
```
- If `"result": "updated"` → Document is modified.
- If `"result": "noop"` → Document was not updated (value unchanged).

## ✏ Update a Document by Adding a New Field
```http
POST /products/_update/100
{
    "doc": {
        "tags" : ["electronics"]
    }
}
```

```http
GET /products/_doc/100
```

## 📝 Update a Document Using a Script
- **Decrease `in_stock` by 1**
```http
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock--"
    }
}
```

- **Set `in_stock` to 10**
```http
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock = 10"
    }
}
```

- **Decrease `in_stock` by a Dynamic Parameter**
```http
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock -= params.quantity",
        "params": {
            "quantity" : 4
        }
    }
}
```

```http
GET /products/_doc/100
```

## 🔄 Upsert (Update if Exists, Create if Not)
```http
POST /products/_update/101
{
    "script": {
        "source": "ctx._source.in_stock++"
    },
    "upsert" : {
        "name" : "Blender",
        "price" : 399,
        "in_stock" : 5
    }
}
```

```http
GET /products/_doc/101
```

## How Elasticsearch Update Document

Elasticsearch me **documents immutable** hote hai, iska matlab hai ki ek baar document index ho gaya to usko directly modify nahi kar sakte. But fir bhi hum **update** kaise karte hai? Yeh ek trick hai! 🚀  

### 🔄 **Elasticsearch me Update ka Process**  
Jab bhi tum ek document update karte ho, Elasticsearch backend me yeh steps follow karta hai:  

1️⃣ **Existing Document ko Retrieve karta hai**  
   - Pehle Elasticsearch current version of document ko fetch karta hai.  

2️⃣ **New Updated Version Create hota hai**  
   - Ek **naya document** create hota hai jo updated values contain karta hai.  

3️⃣ **Old Document ko "Soft Delete" Mark karta hai**  
   - Purane document ko **delete flag** lag jata hai, yani wo visible nahi hota but disk pe hota hai.  

4️⃣ **New Document ko Re-Index karta hai**  
   - Updated document **re-index** hota hai aur naye version ke sath cluster me store ho jata hai.  

5️⃣ **Segment Merge hone ke baad Purana Document Remove hota hai**  
   - Background process me **segment merging** hoti hai jo old soft deleted documents ko permanently remove kar deti hai.  

---


## 🔄 Replace a Document Completely
```http
PUT /products/_doc/100
{
    "name" : "Toaster",
    "price" : 40,
    "in_stock" : 5
}
```

```http
GET /products/_doc/100
```

## 🗑 Delete a Document
```http
DELETE /products/_doc/101
```

```http
GET /products/_doc/101
```

---

**`ctx.op`** Elasticsearch ke scripted updates me **operation control** ke liye use hota hai.  

### **📌 `ctx.op` ke Possible Values**  
1. **`"index"`** → Document ko update ya create karega.  
2. **`"delete"`** → Document ko delete karega.  
3. **`"noop"`** → Kuch bhi change nahi karega (No Operation).  

---

### **📌 Example**

#### **1️⃣ No Operation (`noop`) if `in_stock == 0`**
```json
POST /products/_update/100
{
    "script": {
        "source": """
            if (ctx._source.in_stock == 0) {
                ctx.op = 'noop';
            }
            ctx._source.in_stock--;
        """
    }
}
```
🔹 **Agar stock `0` hai to kuch nahi hoga (`noop`).**  
🔹 **Warna `in_stock` value kam ho jayegi.**  

---

#### **2️⃣ Decrement `in_stock` only if it's greater than 0**
```json
POST /products/_update/100
{
    "script": {
        "source": """
            if (ctx._source.in_stock > 0) {
                ctx._source.in_stock--;
            }
        """
    }
}
```
🔹 **Agar `in_stock` greater than `0` hai tabhi decrement hoga.**  

---

#### **3️⃣ Delete Document if Stock is 1 or Less**
```json
POST /products/_update/100
{
    "script": {
        "source": """
            if (ctx._source.in_stock <= 1) {
                ctx.op = 'delete';
            }
            ctx._source.in_stock--;
        """
    }
}
```
🔹 **Agar stock `1` ya usse kam hai to document delete ho jayega.**  
🔹 **Warna sirf `in_stock` value kam hogi.**  

---

### **🎯 Conclusion**  
✔ **`ctx.op = 'delete'`** → Document delete ho jayega.  
✔ **`ctx.op = 'noop'`** → Document me koi change nahi hoga.  
✔ **By default, update operation hoti hai** agar `ctx.op` specify na karo.  