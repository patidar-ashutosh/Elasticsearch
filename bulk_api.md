# **🔹 Bulk API in Elasticsearch**  

Elasticsearch ki **Bulk API** ek powerful feature hai jo **multiple operations** ko ek hi request me execute karne ki facility deti hai. **Bulk API ka use performance optimization ke liye kiya jata hai**, taaki **multiple indexing, updating, deleting operations ek sath efficiently perform kiya ja sake.**  

> 💡 **Matlab:** Agar tumhe ek-ek request bhejne ki jagah ek hi request me **bahut saare documents insert/update/delete** karne ho to **Bulk API sabse efficient option hai.**  

---

## **🔹 Why Use Bulk API? (Advantages)**
✅ **Faster Performance** → Har document ke liye alag request bhejne se acha ek hi request me sab kuch bhejo  
✅ **Less Network Overhead** → Request-response cycle kam hoti hai, latency reduce hoti hai  
✅ **Better Resource Utilization** → Bulk operations **Elasticsearch ke shards aur nodes ka better utilization** karte hain  
✅ **Efficient for Large Data Sets** → Agar bahut saare documents ko ek sath process karna ho, to best approach hai  

---

## **🔹 Bulk API Syntax**
Bulk API me **JSON Lines (newline-delimited JSON format)** ka use hota hai. Har operation ka structure **2 lines ka hota hai**:

1. **Meta-data Line** (Specify operation: index, create, update, delete)  
2. **Document Data** (Only for `index`, `create`, `update` me hota hai. `delete` me nahi hota)  

```http
POST /_bulk
{ "operation": { "metadata" } }
{ "document data" }
{ "operation": { "metadata" } }
{ "document data" }
```
---

## **1️⃣ Bulk Indexing (Insert Multiple Documents in One Request Using Index)**  
Agar tumhe **multiple documents ek sath insert karne ho**, to **`index` operation** ka use hota hai.  

```http
POST /_bulk
{ "index": { "_index": "products", "_id": "1" } }
{ "name": "Laptop", "price": 800, "category": "electronics" }
{ "index": { "_index": "products", "_id": "2" } }
{ "name": "Smartphone", "price": 500, "category": "electronics" }
{ "index": { "_index": "products", "_id": "3" } }
{ "name": "Microwave", "price": 150, "category": "kitchen" }
```

✔ **Yeh request ek sath 3 documents ko `products` index me insert karegi.**  

📌 **Meta-data line:** `{ "index": { "_index": "products", "_id": "1" } }`  
📌 **Document line:** `{ "name": "Laptop", "price": 800, "category": "electronics" }`  

✔ **Har document ka ek meta-data line aur ek data line hota hai.**  

---

## **2️⃣ Bulk Indexing (Insert Multiple Documents in One Request Using Create)**  

**Bulk API** me `"create"` operation bhi hota hai, jo **sirf naye documents insert** karne ke liye use hota hai. `"index"` bhi documents insert karta hai, but **`"index"` agar document pehle se exist karta ho to usko overwrite kar deta hai**.  

Agar tumhe ensure karna hai ki **documents sirf naye hi create ho, existing documents overwrite na ho**, to `"create"` operation ka use karna chahiye.  

```http
POST /_bulk
{ "create": { "_index": "products", "_id": "10" } }
{ "name": "Air Fryer", "price": 120, "category": "kitchen" }
{ "create": { "_index": "products", "_id": "11" } }
{ "name": "Vacuum Cleaner", "price": 200, "category": "home" }
```

✔ **Yeh request ID `10` aur `11` wale naye documents create karegi.**  
❌ **Agar ID `10` ya `11` ka document pehle se exist karega, to error aayega aur wo overwrite nahi hoga.**  

---

🚀 **Difference Between `index` and `create` in Bulk API**  
| Operation  | Document Already Exists? | Action Taken |
|------------|-----------------|-------------|
| `index`    | ❌ No           | ✅ New document create hoga |
| `index`    | ✅ Yes          | 🔄 Existing document overwrite hoga |
| `create`   | ❌ No           | ✅ New document create hoga |
| `create`   | ✅ Yes          | ❌ Error aayega (document overwrite nahi hoga) |

---

## **3️⃣ Bulk Update (Updating Multiple Documents in One Request)**
Agar tumhe **multiple documents ko update karna ho**, to **`update` operation** ka use hota hai.

```http
POST /_bulk
{ "update": { "_index": "products", "_id": "1" } }
{ "doc": { "price": 850 } }
{ "update": { "_index": "products", "_id": "2" } }
{ "doc": { "category": "smart devices" } }
```

✔ **Yeh request ek sath do documents update karegi:**  
🔹 **ID: `1`** → `"price": 850` ho jayega  
🔹 **ID: `2`** → `"category": "smart devices"` ho jayega  

📌 **Note:** **`doc` ke andar sirf wahi fields likho jo update karni hain.**  

---

## **4️⃣ Bulk Delete (Deleting Multiple Documents in One Request)**
Agar tumhe **multiple documents ko ek sath delete karna ho**, to **`delete` operation** ka use hota hai.

```http
POST /_bulk
{ "delete": { "_index": "products", "_id": "1" } }
{ "delete": { "_index": "products", "_id": "3" } }
```

✔ **Yeh request ID `1` aur `3` wale documents ko delete karegi.**  
📌 **Delete operation ke case me koi second line nahi hoti, kyunki document ka data pass nahi karna hota.**  

---

## **5️⃣ Mixed Bulk Operations (Insert + Update + Delete in One Request)**
Bulk API me tum **mix operations bhi perform kar sakte ho**, jisme **kuch documents insert, kuch update aur kuch delete ho sakte hain ek hi request me.**  

```http
POST /_bulk
{ "index": { "_index": "products", "_id": "4" } }
{ "name": "Blender", "price": 80, "category": "kitchen" }
{ "update": { "_index": "products", "_id": "2" } }
{ "doc": { "price": 550 } }
{ "delete": { "_index": "products", "_id": "3" } }
```

✔ **Yeh ek sath 3 alag operations perform karega:**  
✅ **ID `4` ka document insert hoga**  
✅ **ID `2` ka `price` update hoga**  
✅ **ID `3` ka document delete hoga**  

---

## **🔹 Error Handling in Bulk API**
Agar **kisi ek operation me error aati hai, to bulk API sirf usi operation ko fail karegi**, baki ke operations successful rahenge.  

Bulk API ka response kuch is tarah ka hota hai:  

```json
{
  "took": 30,
  "errors": false,
  "items": [
    {
      "index": {
        "_index": "products",
        "_id": "1",
        "status": 201
      }
    },
    {
      "update": {
        "_index": "products",
        "_id": "2",
        "status": 200
      }
    },
    {
      "delete": {
        "_index": "products",
        "_id": "3",
        "status": 200
      }
    }
  ]
}
```

✔ **`errors: false` ka matlab hai ki sab kuch successfully execute ho gaya.**  
✔ **Agar koi error hoti to `errors: true` hota aur `status` me 400/404 type ka code hota.**  

---

## **🔹 When to Use Bulk API?**
✔ **Jab tum ek sath multiple documents insert/update/delete karna chahte ho**  
✔ **Jab tum performance optimize karna chahte ho**  
✔ **Jab tum ElasticSearch ke request-response overhead ko kam karna chahte ho**  
✔ **Jab tum large data sets ko efficiently process karna chahte ho**  

---

## **🔹 Key Takeaways**
✅ **Bulk API ek sath multiple operations perform karne ka sabse fast aur efficient tarika hai.**  
✅ **Yeh request-response cycles ko reduce karta hai, jo performance optimize karta hai.**  
✅ **Bulk API me har operation ke liye ek meta-data line hoti hai.**  
✅ **Agar kisi ek operation me error ho to bhi baki operations execute hote hain.**  
✅ **Iska use large-scale data processing ke liye best practice hai.**  

💡 **Ab agar tumhe ek sath bahut saare documents insert/update/delete karne ho, to hamesha `Bulk API` ka use karo!** 🚀

---
[Reference Video Link](https://youtu.be/SKY_oYz6628?si=HjIM4-e6rIQFI0Dy)
---