---
title: "Factory vs. Repository ב-Domain Driven Design: מה ההבדל ולמה זה קריטי?"
datePublished: 2025-08-17T00:00:00.000Z
cuid: cmi7ek8sp000702l5bfm70sp3
slug: factory-vs-repository-v-domain-driven-design-mh-hhvdl-vlmh-zh-kryty

---


### אמ;לק

בקצרה, **Factory** אחראי על יצירת אובייקטים או Aggregates מורכבים במצב התחלתי תקין. **Repository** אחראי על שליפה ושמירה של Aggregates קיימים ממנגנון persistence (כמו database), ויוצר אשליה שה-Aggregate קיים בזיכרון. הראשון הוא נקודת ההתחלה של אובייקט, השני מנהל את מחזור החיים שלו לאחר היצירה.

* * *

### הקדמה

[בפוסט הקודם](https://www.thecodeline.org/ddd/) בסדרת ה-DDD, דיברנו על החשיבות של Aggregates כאחת מאבני הבניין המרכזיות במידול הדומיין. הגדרנו אותם כקפסולה של אובייקטים (Entities, Value Objects) שמתקיימת כיחידה עסקית אחת עם חוקים (Invariants) ברורים.

זה מעולה אבל עכשיו עולות שתי שאלות פרקטיות לקריטיות:

1. איך אנחנו יוצרים Aggregate מורכב בצורה נכונה, כך שיכבד את כל החוקים העסקיים שלו מהרגע הראשון?

3. איך אנחנו שומרים ושולפים את ה-Aggregates האלה מבסיס הנתונים, בלי לזהם את הלוגיקה העסקית שלנו בקוד תשתיתי?

כאן נכנסים לתמונה שני Patterns חיוניים בעולם ה-DDD: ה-**Factory** וה-**Repository**. רבים מתבלבלים ביניהם, אבל ההבחנה ביניהם היא המפתח לכתיבת קוד נקי, מובן וקל לתחזוקה. בואו נצלול לעומק

* * *

### דפוס ה-Factory: מלאכת היצירה

התפקיד של Factory ב-DDD הוא מאוד ספציפי: **להיות אחראי על יצירת אובייקטים מורכבים או Aggregates שלמים.** למה אנחנו צריכים את זה? כי לפעמים תהליך היצירה וא יותר מסתם קריאה ל-`new Object()`.

חשבו על Aggregate של `Order` (הזמנה). יצירת הזמנה חדשה עשויה לכלול:

- יצירה מזהה יחודי (ID).

- הוספת פריט ראשון לסל (הזמנה לא יכולה להתקיים ללא פריטים).

- קביעת סטאטוס התחלתי (למשל ״Pending").

- בדיקה שהלקוח המזמין הוא לקוח פעיל.

כל אלו הם חוקים עסקיים (Invariants) שחייבים להתקיים מרגע _היצירה_. דיחסת כל הלוגיקה הזו לתוך ה-constructor של `Order` יכולה לסבך אותנו מאוד. ה-Factory מאפשר לנו לרכז את הלוגיקה הזו במקום אחד, נקי וברור, ולהבטיח שכל `Order` שנוצר במערכת הוא תקין לחלוטין.

Kotlin

```
import java.util.UUID
<div></div>
// Value Objects for Type Safety
data class OrderId(val value: String)
data class CustomerId(val value: String)
<div></div>
// Other domain classes...
data class Product(val id: String, val price: Double, private val inStock: Boolean) {
    fun isInStock() = inStock
}
data class Customer(val id: CustomerId, private val active: Boolean) {
    fun isActive() = active
}
data class LineItem(val productId: String, val price: Double)
<div></div>
enum class OrderStatus {
    PENDING, SHIPPED, CANCELED
}
<div></div>

class Order private constructor(
    val id: OrderId,
    val customerId: CustomerId,
    initialItems: List<LineItem>
) {
    private val _lineItems: MutableList<LineItem> = initialItems.toMutableList()
    val lineItems: List<LineItem> get() = _lineItems.toList()
<div></div>
    var status: OrderStatus = OrderStatus.PENDING
        private set
<div></div>
    companion object {
        fun createNew(customer: Customer, initialItem: Product): Order {
           
            // Enforcing invariants (business rules)
            require(customer.isActive()) { "Cannot create order for an inactive customer." }
            require(initialItem.isInStock()) { "Cannot create order with an out-of-stock item." }
<div></div>
            val newOrderId = OrderId(UUID.randomUUID().toString())
            val firstLineItem = LineItem(initialItem.id, initialItem.price)
            
            return Order(newOrderId, customer.id, listOf(firstLineItem))
        }
    }
<div></div>
    fun ship() {
        if (this.status == OrderStatus.PENDING) {
            this.status = OrderStatus.SHIPPED
        }
    }
}
```

```
import java.util.UUID

// Value Objects for Type Safety
data class OrderId(val value: String)
data class CustomerId(val value: String)

// Other domain classes...
data class Product(val id: String, val price: Double, private val inStock: Boolean) {
    fun isInStock() = inStock
}
data class Customer(val id: CustomerId, private val active: Boolean) {
    fun isActive() = active
}
data class LineItem(val productId: String, val price: Double)

enum class OrderStatus {
    PENDING, SHIPPED, CANCELED
}

class Order private constructor(
    val id: OrderId,
    val customerId: CustomerId,
    initialItems: List<LineItem>
) {
    private val _lineItems: MutableList<LineItem> = initialItems.toMutableList()
    val lineItems: List<LineItem> get() = _lineItems.toList()

    var status: OrderStatus = OrderStatus.PENDING
        private set

    companion object {
        fun createNew(customer: Customer, initialItem: Product): Order {
           
            // Enforcing invariants (business rules)
            require(customer.isActive()) { "Cannot create order for an inactive customer." }
            require(initialItem.isInStock()) { "Cannot create order with an out-of-stock item." }

            val newOrderId = OrderId(UUID.randomUUID().toString())
            val firstLineItem = LineItem(initialItem.id, initialItem.price)
            
            return Order(newOrderId, customer.id, listOf(firstLineItem))
        }
    }

    fun ship() {
        if (this.status == OrderStatus.PENDING) {
            this.status = OrderStatus.SHIPPED
        }
    }
}
```

בדוגמה הזו, ה-`Factory` הוא מתודה סטטית `createNew` על האובייקט `Order` עצמו. היא מבטיחה שאף אחד לא יכול ליצור `Order` במצב לא חוקי.

* * *

### דפוס ה-Repository: שער לעולם הדאטה

אחרי שיצרנו `Order` תקין, אנחנו רוצים לשמור אותו. וחשוב יותר, אנחנו רוצים להיות מסוגלים למצוא אותו אחר-כך. כאן ה-Repository נכנס לתמונה.

התקפיד של **Repository** הוא לגשר בין המודל של הדומיין (ה-Aggregates) לבין מנגנון ה-persistence (בסיס נתונים, קבצים, שירות חיצוני וכו׳). הוא עושה זאת על ידי יצירת **אשליה של קולקציה בזיכרון**. הקוד העסקי שלנו לא צריך לדעת אם הנתנונים מגיעים מ-MongoDB, PostgreSQL או קובץ CSV. הוא פשוט מבקש מה-Repository את ה-`Order` שהוא צריך.

ל-Repository יש בדרך כלל מתדות פשוטות וברורות:

- `getById(id)`: מחזיר Aggregate ספציפי

- `save(aggregate)`: שמור Aggregate (בין אם חדש או קיים).

- `findBySomeCriteria(...)`: מחזיר רשימה של Aggregates שעונים על תנאי מסוים.

Kotlin

```
interface OrderRepository {
  fun findById(orderId: OrderId): Order?
  fun save(order: Order)
  fun findPendingOrdersForCustomer(customerId: CustomerId): List<Order>
}
```

```
interface OrderRepository {
  fun findById(orderId: OrderId): Order?
  fun save(order: Order)
  fun findPendingOrdersForCustomer(customerId: CustomerId): List<Order>
}
```

* * *

### ההבדל הקריטי, והסינרגיה ביניהם

אז מה ההבדל המהותי?

| **מאפיין** | Factory | Repository |
| --- | --- | --- |
| **אחריות** | יצירת אובייקטים **חדשים** במצב תקין | שליפה ושמירה של אובייקטים **קיימים** |
| **קלט** | נתוני גלם, אובייקטים אחרים | מזהה (ID) או קריטריון חיפוש |
| **פלט** | Aggregate חדש, שמעולם לא נשמר | Aggregate שכבר קיים (או רשימה) |
| **תחום עניין** | תחילת מחזור החיים | אמצע וסוף מחזור החיים |

ההבנה של ההבדל בין Factory ל-Repository היא צעד משמעותי בדרך לכתיבת קוד DDD איכותי.

- השתמשו ב-**Factory** כאשר היצירה של אובייקט היא תהליך מורכב עם חוקים עסקיים, כדי להבטיח ולידציה מלאה מרגע הלידה.

- השתמשו ב-**Repository** כדי להפריד באופן מוחלט בין הלוגיקה העסקית לבין אופן שמירת הנתונים ולנהל את מחזור החיים של ה-Aggregates שלכם.

ההפרדה הזו לא רק עושה סדר בקוד; היא מאפשרת גמישות ארכיטקטונית, מקלה על בדיקות, והופכת את הדומיין העסקי שלכם לברור ונהיר הרבה יותר.
