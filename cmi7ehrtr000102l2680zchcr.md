---
title: "פאטרן Acyclic Visitor: לשבור את מעגל התלות של Visitor"
datePublished: 2025-08-03T00:00:00.000Z
cuid: cmi7ehrtr000102l2680zchcr
slug: ftrn-acyclic-visitor-lvvr-t-mgl-htlvt-l-visitor

---


בעולם של תכנון תוכנה, Design Patterns הם כמו מתכונים בדוקים לבעיות נפוצות. פאטרן ה-Visitor הוא אחד הכלים החזקים בארגז הכלים שלנו, אבל כמו כל כלי, יש לו נקודות חולשה. היום נצלול לעומק אחת מהן, ונכיר את האח הפחות מפורסם אך הגמיש להפליא שלו: ה-Acyclic Visitor.

### אמ;לק

פאטרן ה-Visitor הקלאסי מאפשר להוסיף פעולות חדשות על קבוצת אובייקטים מבלי לשנות את האובייקטים עצמם. הבעיה הגדולה שלו היא שהוא מאוד שביר כאשר רוצים להוסיף _אובייקט_ מסוג חדש למערכת; שינוי כזה מכריח עדכון של כל ה-Visitors הקיימים. פאטרן **Acyclic Visitor** פותר את הבעיה הזו על ידי הפיכת כיוון התלות. הוא מאפשר להוסיף אובייקטים חדשים למערכת מבלי לשנות קוד קיים, במיחר של ויתור על חלק מהבדיקות של הקומפיילר והסתמכות על בדיקות בזמן ריצה.

### למה בכלל צריך Visitor? תזכורת על הפאטרן הקלאסי

בואו ניישר קו. דמיינו מערכת לניהול **נכסים** (`Assets`) של אזרח במדינה. יש לנו קלאסים כמו `Apartment`, `Car`, `Salary`.

עכשיו, גופים שונים רוצים לחשב מיסים על הנכסים הללו.

- **הדרך הנאיבית**: להוסיף לכל קלאס מתודות כמו `calculateArnonna()`, `calculateIncomeTax()` וכו׳. זה יהפוך את הקלאסים למסורבלים ויפר את העקרון האחריות הבודדת.

- **דרך ה-Visitor**: אנחנו מפרידים את **הפעולה** (חישוב המס) מה**אובייקט** (הנכס). ניצור `Visitor` לכל גוף, למשל, `IncomeTaxVisitor`. ה-Visitor הזה ״יבקר״ כל נכס ויבצע את החישוב הרלוונטי עבורו.

היתרון ברור: אם מחר נרצה להוסיף תמיכה בביטוח לאומי או מס חדש, פשוט ניצור Visitor חדש. הקוד של הנכסים לא משתנה כלל.

* * *

### הסדק ביסודות - הבעיה המרכזית בפאטרן Visitor הקלאסי

אז איפה הבעיה? הבעיה מתגלה כשהמערכת שלנו צריכה לגדול לא בכמות הפעולות, אלא בכמות האובייקטים.

נניח שעכשיו הכנסת מחליטה להכיר ולהטיל מס על נכס מסוג חדש: `CryptocurrencyWallet` (ארנק קריפטו).

כדי שה-Visitors שלנו יכירו את הנכס החדש, אנחנו חייבים לעדכת את האינטרפייס הראשי `Visitor`:

Kotlin

```
// Before
interface Visitor {
  fun visit(apt: Apartment)
  fun visit(salary: Salary)
  fun visit(car: Car)
}
<div></div>
// After adding new asset
interface Visitor {
    fun visit(apt: Apartment)
    fun visit(salary: Salary)
    fun visit(cw: CryptocurrencyWallet) //this line breaks all implements
}
```

```
// Before
interface Visitor {
  fun visit(apt: Apartment)
  fun visit(salary: Salary)
  fun visit(car: Car)
}

// After adding new asset
interface Visitor {
    fun visit(apt: Apartment)
    fun visit(salary: Salary)
    fun visit(cw: CryptocurrencyWallet) //this line breaks all implements
}
```

ברגע שהוספנו את `visit(CryptocurrencyWallet)`, הקומפיילר יתחיל לצעוק. **כל** ה-Visitors הקיימים שלנו - `ArnonaVisitor`, `IncomeTaxVisitor` וכל השאר - שבורים! עכשיו אנחנו חייבים לעבור על כולם ולממש את המתודה החדשה, גם אם לגוף כמו עירייה אין שום עניין או קשר לקריפטו.

זוהי **תלות מעגלית** (Cyclic Dependency). הוספת `Element` חדש גורמת לאפקט אדווה שובר בכל המערכת.

* * *

### הפתרון - הכירו את Acyclic Visitor

כאן נכנס לתמונה הפתרון של רוברט מרטין (Uncle Bob). הרעיון המרכזי הוא לשבור את התלות הזו על ידי שימוש ב-[Interface Segregation Principle](https://www.thecodeline.org/isp/) ובהיפוך תלויות.

במקום איטרפייס `Visitor` אחד גדול שמכיר את כל הנכסים נעשה כך:

- ניצור אינטרפייס `Visitor` בסיסי ו**ריק** (Marker Interface).

- כל קלאס `Asset` (למשל `Apartment`) יגדיר אינטרפייס `Visitor` קטן וספציפי משלו (למשל `ApartmentVisitor`)

- ה-`Concrete Visitors` שלנו יממשו את אינטרפייסי ה-Visitors הספציפיים של הנכסים שבהם הם מעוניינים לטפל.

כך, היררכיית הנכסים כבר לא תלויה ב-Visitor, אלא ה-Visitor תלוי בה. שברנו את המעגל.

### איך הקסם עובד? מבט לעומק המימוש

הקסם מתבצע באמצעות מנגנון שנקרא **Double Dispatch** בשילוב עם בדיקת טיפוסים בזמן ריצה. ה-`accept` של כל נכס יקבל `Visitor` בסיסי, אבל ינסה להמיר אותו (באמצעות `dynamic_cast`) לטיפוס ה-Visitor הספציפי שלו.

- אם ההמרה מצליחה  -> סימן שה-Visitor הזה יודע לטפל בנכס הזה, ואפשר לקרוא למתודת ה-`visit` שלו.

- אם ההמרה נכשלת -> סימן שה-Visitor לא מכיר את הנכס הזה. פשוט לא עושים כלום.

בואו נראה את זה בקוד זה יבהיר הכל.

### בואו נכתוב קוד - דוגמה מעשית

התיאוריה חשובה, אבל אין כמו לראות קוד כדי להבין באמת. בואו נממש את תרחיש הנכסים והמסים שלנו באמצעות Acyclic Visitor

#### שלב א׳: הגדרת מחלקות בסיסית

Kotlin

```
// Empty and basic Visitor. Use for marker
interface Visitor
<div></div>
// Basic interface for all assets
interface Asset {
  fun accept(visitor: Visitor)
}
```

```
// Empty and basic Visitor. Use for marker
interface Visitor

// Basic interface for all assets
interface Asset {
  fun accept(visitor: Visitor)
}
```

#### שלב ב׳: הגדרת הנכסים (Elements) וה-Visitors הספציפיים להם

Kotlin

```
class Apartment : Asset {
  
  // Specific visitor interface for the Apartment asset
  interface ApartmentVisitor : Visitor {
    fun visit(apt: Apartment)
  }
  
  override fun accept(visitor: Visitor) {
    // The magic: 
    if (visitor is ApartmentVisitor) {
      visitor.visit(this)
    }
  }
}
<div></div>
class StockPortfolio : Asset {
<div></div>
  interface StockPortfolioVisitor : Visitor {
    fun visit(sp: StockPortfolio)
  }
  
  override fun accept(visitor: Visitor) {
    if (visitor is StockPortfolioVisitor) {
      visitor.visit(this)
    }
  }
}
<div></div>
// Implemnt other assets
```

```
class Apartment : Asset {
  
  // Specific visitor interface for the Apartment asset
  interface ApartmentVisitor : Visitor {
    fun visit(apt: Apartment)
  }
  
  override fun accept(visitor: Visitor) {
    // The magic: 
    if (visitor is ApartmentVisitor) {
      visitor.visit(this)
    }
  }
}

class StockPortfolio : Asset {

  interface StockPortfolioVisitor : Visitor {
    fun visit(sp: StockPortfolio)
  }
  
  override fun accept(visitor: Visitor) {
    if (visitor is StockPortfolioVisitor) {
      visitor.visit(this)
    }
  }
}

// Implemnt other assets
```

#### שלב ג׳: מימוש גופי המיסוי (Concrete Visitors)

Kotlin

```
// The IRS is interesetd in the Salary and StockProtfolio
class IncomeTaxVisitor : Salary.SalaryVisitor, StockPortfolio.StockPortfolioVisitor {
  
  override fun visit (sl: Salary) {
    println("IRS: calculate tax for salary")
  }
  
  override fun visit(sp: StockPortfolio) {
    println("IRS: calculate tax for stock portfolio")
  }
}
```

```
// The IRS is interesetd in the Salary and StockProtfolio
class IncomeTaxVisitor : Salary.SalaryVisitor, StockPortfolio.StockPortfolioVisitor {
  
  override fun visit (sl: Salary) {
    println("IRS: calculate tax for salary")
  }
  
  override fun visit(sp: StockPortfolio) {
    println("IRS: calculate tax for stock portfolio")
  }
}
```

#### שלב ד׳: רגע האמת - הוספה של נכס חדש למערכת

Kotlin```
class CryptocurrencyWallet : Asset {
  interface CryptoVisitor : Visitor {
    fun visit(cw: CryptoCurrencyWallet)
  }
  
  override fun accept(visitor: Visitor) {
    if (visitor is CryptoVisitor) {
      visitor.visit(this)
    }
  }
}

```

```
class CryptocurrencyWallet : Asset {
  interface CryptoVisitor : Visitor {
    fun visit(cw: CryptoCurrencyWallet)
  }
  
  override fun accept(visitor: Visitor) {
    if (visitor is CryptoVisitor) {
      visitor.visit(this)
    }
  }
}
```

עכשיו נעדכן את ה- `IncomeTaxVisitor` כדי שיתחשב גם ב-asset החדש

Kotlin

```
class IncomeTaxVisitor : Salary.SalaryVisitor, StockPortfolio.StockPortfolioVisitor, CryptocurrencyWallet.CryptoVisitor {
  
  override fun visit (sl: Salary) {
    println("IRS: calculate tax for salary")
  }
  
  override fun visit(sp: StockPortfolio) {
    println("IRS: calculate tax for stock portfolio")
  }
  
  // new
  override fun visit(vw: CryptocurrencyWallet) {
    println("IRS: calculate tax for crypto profits")
  }
}
```

```
class IncomeTaxVisitor : Salary.SalaryVisitor, StockPortfolio.StockPortfolioVisitor, CryptocurrencyWallet.CryptoVisitor {
  
  override fun visit (sl: Salary) {
    println("IRS: calculate tax for salary")
  }
  
  override fun visit(sp: StockPortfolio) {
    println("IRS: calculate tax for stock portfolio")
  }
  
  // new
  override fun visit(vw: CryptocurrencyWallet) {
    println("IRS: calculate tax for crypto profits")
  }
}
```

#### שלב ה׳: איך הכל מתחבר

Kotlin

```
fun main() {
  val assets: List<Asset> = listof(Apartment(), StockPortfolio(), Salary(), CryptocurrencyWallet())
  
  val arnona = ArnonaVisitor()
  for (asset in assets) {
    asset.accept(arnona) // print only for Apatment
  }
  
  val incomeTax = IncomeTaxVisitor()
  for (asset in assets) {
    asset.accept(incomeTax) // print for all assets beside Apartment
  }
}
```

```
fun main() {
  val assets: List<Asset> = listof(Apartment(), StockPortfolio(), Salary(), CryptocurrencyWallet())
  
  val arnona = ArnonaVisitor()
  for (asset in assets) {
    asset.accept(arnona) // print only for Apatment
  }
  
  val incomeTax = IncomeTaxVisitor()
  for (asset in assets) {
    asset.accept(incomeTax) // print for all assets beside Apartment
  }
}
```

* * *

### ניתוח - יתרונות, חסרונות וטרייד-אופס

שום דבר לא בא בחינם, וחשוב להבין את הטרייד-אופס

#### יתרונות

- **גמישות מקסימאלית**: ניתן להוסיף `Element` חדשים מבלי לשבור קוד קיים.

- **שבירת תלות מעגלית**: אין יותר קשר הדוק בין היררכיית ה-Elements ל-Visitors.

- **עמידה בעקרות SOLID**: ביעקר ב-Open/Close Principle וב-[Interface Segregation Principle](https://www.thecodeline.org/isp/).

#### חסרונות

- **אובדן Compile-Time Saftey**: בפאטרן הקלאסט, הקומפיילר הבטיח שהכל מכוסה. כאן, טעות יכולה להתגלות רק בזמן ריצה.

- **מורכבות ו-Boilerplate**: המימוש דורש יותר קוד תבניתי ואינטרפייסים.

- **עלות ביצועים (זניחה לרוב)**: ל-`dynamic_cast` יש עלות זמן ריצה קטנה.

* * *

### סיכום - הנקודות המרכזיות לקחת הביתה

הבחירה בין Visitor קלאסט ל-Acyclic Visitor היא החלטה ארכיטקטונית שתלויה במה שאתם צופים שישתנה במערכת שלכם בעתיד.

- אם רשימת ה**אלמנטים** שלכם יציבה, אבל אתם צופים שיתווספו **פעולות** חדשות - ה-Visitor הקלאסי הוא פתרון מצוין, בטוח ופשוט.

- אם רשימת **הפעולות** יציבה, אבל אתם צופים שיהיו **אלמנטים** חדשים במערכת שלכם ה- Acyclic Visitor הוא פתרון עמיד, גמיש ונכון יותר בטווח הארוך.

בפעם הבאה שאתם שוקלים להשתמש ב-Visitor, שאלו את עצמכם: ״מה צפוי להשתנות יותר, הפעולות או האובייקטים?״.  
התשובה לשאלה הזו תוביל אתכם לבחירה הנכונה.
