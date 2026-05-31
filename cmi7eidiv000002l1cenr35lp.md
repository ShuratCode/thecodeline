---
title: "מ-Fat Interface ל-Fine Interface: איך עקרון ה-ISP ישדרג לכם את הארכיטקטורה"
datePublished: 2025-08-10T00:00:00.000Z
cuid: cmi7eidiv000002l1cenr35lp
slug: m-fat-interface-l-fine-interface-ykh-krvn-h-isp-ydrg-lkhm-t-hrkhytktvrh

---


### אמ;לק

- **הבעיה:** ממשקים ״שמנים״ (Fat Interfaces) עם המון מתודות מכריחים מחלקות לממש פונקציונליות שהן לא צריכות. זה יותר קוד שביר, מסורבל וקשה לתחזוקה.

- **הפתרון:** עקרון הפרדת ממשקים (ISP) אומר: ״אל תכריחו לקוחות להיות תלויים בממשקים שהם לא משתמשים בהם״. במקום ממשק אחד גדול, ניצור מספר ממשקים קטנים וממוקדי יכולת.

- **הרווח**: קוד נקי יותר, גמיש, עם תלויות נמוכות (low coupling) שקל הרבה יותר להרחיב ולתחזק לאורך זמן

* * *

### פתיח

כולנו נתקלנו בזה: ממשק `IWhateverService` עם 15 מתודות, שחצי מהן בכלל לא רלוונטיות למחלקה החדשה שאנחנו כותבים. מה עושים? ממשים אותן כמתודות ריקות או זורקים `NoImplementedException`?  
התופעה הזאת, המכונה ״ממשק שמן״ (Fat Interface), היא מחלק שקטה שמסבכת את הקוד, מגבירה תלויות (coupling) והופכת את התחזוקה לסיוט. בדיוק בשביל לפתור את הבעיה הזו קיים עקרון ה- Interface Segregation Principle (ISP), אחד מחמש עקרונות ה-SOLID שכל מפתח חייב להכיר.

### הבעיה: ממשקים ״שמנים״ (Fat Interaces)

ממשק ״שמן״ הוא ממשק המכיל מספר רב של מתודות, אשר אינן קשורות כולן באופן הדוק או שאינן נחוצות לכל ה-clients הצורכים את הממשק הזה. הבעיה היעקרית בממשקים ״שמנים״ היא שהם כופים על מחלקות ליישם מתודות שהן אינן זקוקות להן. הדבר עלול לגרום לקוד מיותר ומטעה, כמו יישום מתודות שפשוט זורקות `Exception` או נשארות ריקות.

מעבר לכך, ממשקים ״שמנים״ יוצרים coupling לא רצוי בין רכיבים שונים. אם לקוח אחד דורש שינוי בחתימה של מתודה מסוימת בממשק, כל שאר הלקוחות התלויים באותו ממשק ייאלצו לעבור קומפילציה מחדש או אף שינויים בקוד שלהם, גם אם המתודה שהשתנתה אינה רלוונטית עבורם. הדבר הופך את עלויות התחזוקה ואת הסיכונים הכרוכים בשינויים לבלתי צפויים ומטשטש את האחריות האמיתי של כל המחלקה.

### הפתרון: The Interface Segregation Principle (ISP)

עקרון ה-ISP מנוסח בפשטות:

> לקוחות לא צריכים להיות מאולצים להיות תלויים בממשקים שהם אינם משתמשים בהם.

המשמעות היא שעלינו להעדיף הרבה ממשקים קטנים, ממוקדים וספציפיים ללקוח (cohesive) על פני ממשק אחד גדול וכללי. ממשק קוהרנטי מכיל רק מתודות שקשורות זו לזו באופן הדוק ומשרתות מטרה ברורה.

### דוגמת קוד: מתיאוריה לקוד נקי

בואו נראה דוגמה קלאסית בקוטלין.  
דמיינו שאנחנו בונים מערכת גדולה עם microservices. יש לנו צורך לגשת למידע על משתמשים ממקומות שונים במערכת, אבל עם הרשאות וצרכים שונים.

#### לפני ISP: ממשק Repository ״כלבויניק״

הנטייה הראשונית היא ליצור ממשק אחד, `IUserRepository`, שיודע לעשות הכל

Kotlin

```
// Fat interface that contains read, write, and cache management actions
interface UserRepository {
  fun getUserById(id: String): User
  fun findUsersByCountry(countryCode: String): List<User>
  fun saveUser(user: User)
  fun deleteUser(id: String)
  fun invalidateCacheForUser(id: String)
}
```

```
// Fat interface that contains read, write, and cache management actions
interface UserRepository {
  fun getUserById(id: String): User
  fun findUsersByCountry(countryCode: String): List<User>
  fun saveUser(user: User)
  fun deleteUser(id: String)
  fun invalidateCacheForUser(id: String)
}
```

עכשיו בואו נראה איך זה מסתבך. יש לנו שני צרכנים שונים:

1. **שירות דוחות (Reporting Service):** צריך רק לקרוא נתוני משתמשים כדי להפיק דוחות. הוא לעולם לא אמור לכתוב או למחוק מידע.

3. **שירות ניהול משתמשים (User Management Service):** צריך את כל היכולות - קריאה, כתיבה, מחיקה וניהול cache.

כך תיראה הבעיה במימוש של השירות דוחות:

Kotlin

```
class ReportingUserRepository(private val dbConnection: DbConnection) : UserRepository {
<div></div>
    override fun getUserById(id: String): User {
        // logic to read from DB
        println("Fetching user $id for report")
        return User(id, "Test User")
    }
<div></div>
    override fun findUsersByCountry(countryCode: String): List<User> {
        // login to read from DB
        println("Fetching users from $countryCode for report")
        return listOf()
    }
<div></div>
    // The real problem
    override fun saveUser(user: User) {
        throw UnsupportedOperationException("Reporting repository is read-only!")
    }
<div></div>
    override fun deleteUser(id: String) {
        throw UnsupportedOperationException("Reporting repository is read-only!")
    }
<div></div>
    override fun invalidateCacheForUser(id: String) {
       // Leave empty? That is confusing 
    }
}
```

```
class ReportingUserRepository(private val dbConnection: DbConnection) : UserRepository {

    override fun getUserById(id: String): User {
        // logic to read from DB
        println("Fetching user $id for report")
        return User(id, "Test User")
    }

    override fun findUsersByCountry(countryCode: String): List<User> {
        // login to read from DB
        println("Fetching users from $countryCode for report")
        return listOf()
    }

    // The real problem
    override fun saveUser(user: User) {
        throw UnsupportedOperationException("Reporting repository is read-only!")
    }

    override fun deleteUser(id: String) {
        throw UnsupportedOperationException("Reporting repository is read-only!")
    }

    override fun invalidateCacheForUser(id: String) {
       // Leave empty? That is confusing 
    }
}
```

הבעיות כאן ברורות:

- אנחנו כופים על מימוש של read-only לממש (ולזרוק שגיאות) בפעולות כתיבה.

- אנחנו חושפים את שירות הדוחות לפונקציות שהוא לעולם לא צריך להכיר, מה שמגדיל את הסיכון לטעויות.

- מה אם נוסיף לממשק פונקציה נוספת? נצטרך לעדכן את כל המחלקות המממשות, גם אם זה לא רלוונטי אליהן.

#### אחרי ISP: הפרדת אחריות (והרבה שקט נפשי)

במקום הממשק ה״כלבויניק״, נפצל אותו לממשקים קטנים וממוקדי אחריות, בהשראת עקרון CORS (Command Query Responsibility Segregation)

Kotlin

```
// Specific interface for reading actions (Query)
interface UserReader {
  fun getUserById(id: String): User
  fun findUserByCountry(countryCode: String): List<User>
}
<div></div>
// Specific interface for write actions (Command)
interface UserWriter {
  fun saveUser(user: User)
  fun deleteUser(id: String)
}
<div></div>
// Specific interface for manage cache
interface CacheManager {
    fun invalidateCahceForUser(id: String)
}
```

```
// Specific interface for reading actions (Query)
interface UserReader {
  fun getUserById(id: String): User
  fun findUserByCountry(countryCode: String): List<User>
}

// Specific interface for write actions (Command)
interface UserWriter {
  fun saveUser(user: User)
  fun deleteUser(id: String)
}

// Specific interface for manage cache
interface CacheManager {
    fun invalidateCahceForUser(id: String)
}
```

עכשיו המימושים שלנו הופכים להיות נקיים, בטוחים וברורים:

Kotlin

```
// Implement the read-only repo
class ReportingUserRepository(private val dbConnection: DbConnection) : UserReader {
    override fun getUserById(id: String): User {
        println("Fetching user $id for report")
        return User(id, "Test User")
    }
<div></div>
    override fun findUsersByCountry(countryCode: String): List<User> {
        println("Fetching users from $countryCode for report")
        return listOf()
    }
}
<div></div>

//Implement the full repo for the read-write and cache
class FullUserRepository(private val dbConnection: DbConnection) : UserReader, UserWriter, CacheManager {
    
    override fun getUserById(id: String): User { /* ... */ return User(id, "") }
    override fun findUsersByCountry(countryCode: String): List<User> { /* ... */ return listOf() }
    
    override fun saveUser(user: User) { /* ... */ }
    override fun deleteUser(id: String) { /* ... */ }
<div></div>
    override fun invalidateCacheForUser(id: String) { /* ... */ }
}
```

```
// Implement the read-only repo
class ReportingUserRepository(private val dbConnection: DbConnection) : UserReader {
    override fun getUserById(id: String): User {
        println("Fetching user $id for report")
        return User(id, "Test User")
    }

    override fun findUsersByCountry(countryCode: String): List<User> {
        println("Fetching users from $countryCode for report")
        return listOf()
    }
}

//Implement the full repo for the read-write and cache
class FullUserRepository(private val dbConnection: DbConnection) : UserReader, UserWriter, CacheManager {
    
    override fun getUserById(id: String): User { /* ... */ return User(id, "") }
    override fun findUsersByCountry(countryCode: String): List<User> { /* ... */ return listOf() }
    
    override fun saveUser(user: User) { /* ... */ }
    override fun deleteUser(id: String) { /* ... */ }

    override fun invalidateCacheForUser(id: String) { /* ... */ }
}
```

התוצאה היא מערכת טובה משמעותית. שירות הדוחות שלנו יכול להיות תלוי **אך ורק** בממשך `UserReader`, וכך אנחנו מבטיחים ברמת הקומפילציה שהוא לעולם לא יוכל לבצע בטעות פעולת כתיבה או מחיקה. הגדרנו חוזים ברורים, קטנים ובטוחים בין רכיבי המערכת.

### הרווח הנקי: למה זה כל כך משתלם?

יישום עקרון ה-ISP מביא עמו יתרונות משמעותיים:

- **Decoupling:** ה-ISP מפחית באופן דרמטי את ה-coupling הלא רצוי בין הרכיבים. כשממשקים הם קטנים וממוקדים, שינוי בממשק אחד (לצורך client ספציפי) לא ישפיע על לקוחות אחרים התלויים בממשקים השונים.

- **גמישות ותחזוקה:** קוד המבוסס על ממשקים קטנים וקוהרנטיים הוא קל יותר לקריאה, להבנה ולתחזוקה. קל יותר להוסיף פונקציונליות חדשה או לשנות קיימת, מכיוון שאין צורך להתמודד עם מורכבות של ממשקים ״שמנים״.

- **Clarity & Reusability**: הקוד מפחית את כמות הקוד המיותר ומבטיח שכל מתודה בממשק משרתת מטרה ברורה ורלוונטית. זה מוביל בסופו של דבר למערכות יציבות יותר, עם פחות באגים ועם יכולת גבוהה יותר לשימוש חוזר בקוד.

* * *

### סיכום

עקרון ה-Interface Segregation Principle הוא לא רק כלל טכני, אלא שינוי תפיסתי. הוא מאלץ אותנו לחשוב על האבסטרקציות שלנו בצורה ממוקדת יותר ולכנן אותן מנקודת המבט של הלקוח (הקוד שצורך את הממשק).

בפעם הבאה שאתם באים להוסיף מתודה לממשק קיים, עצרו לרגע ושאלו את עצמכם: ״האם כל מי שמממש את הממשק הזה באמת צריך את המתודה החדשה?״. אם התשובה היא ״לא״, כנראה שהגיע הזמן לפצל את הממשק. המאמץ הקטן הזה עכשיו יחסוך לכם המון כאב ראש בתחזוקה ובהרחבת המערכת בעתיד
