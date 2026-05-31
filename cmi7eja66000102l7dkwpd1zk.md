---
title: "if-else-switch"
datePublished: 2023-07-16T00:00:00.000Z
cuid: cmi7eja66000102l7dkwpd1zk
slug: if-else-switch

---


אני הולך לפתוח פה מהומת אלוהים.  
מריבות על code style היא תמיד עניין מלוכלך. לכולם יש דעות על הנושא הזה.  
והרבה שורות כבר נכתבו על רווחים מול טאבים, סוגריים באותה שורה או בשורה חדשה ועוד.  
אחד הנושאים שאוהבים לדון עליהם זה `if-else-switch`.

### המקור לדיון

לפני כמה זמן ראיתי בטוויטר את הדוגמת קוד הבאה

Java

```
// original code
public String determineGender(int input) {
    String gender = "";
    if (input == 0) {
        gender = "male";
    } else if (input == 1) {
        gender = "female";
    } else {
        gender = "unknown";
    }
<div></div>
    return gender;
}
<div></div>
// refactoring 1
public String determineGender(int input) {
    String gender = "";
    switch(input) {
        case 0:
            gender = "male";
            break;
        case 1:
            gender = "female";
            break;
        default:
            gender = "unknown";
            break;
    }
<div></div>
    return gender;
}
<div></div>
// refactoring 2
public String determineGender(int input) {
    if (input == 0) return "male";
    if (input == 1) return "female";
    return "unknown";
}
```

```
// original code
public String determineGender(int input) {
    String gender = "";
    if (input == 0) {
        gender = "male";
    } else if (input == 1) {
        gender = "female";
    } else {
        gender = "unknown";
    }

    return gender;
}

// refactoring 1
public String determineGender(int input) {
    String gender = "";
    switch(input) {
        case 0:
            gender = "male";
            break;
        case 1:
            gender = "female";
            break;
        default:
            gender = "unknown";
            break;
    }

    return gender;
}

// refactoring 2
public String determineGender(int input) {
    if (input == 0) return "male";
    if (input == 1) return "female";
    return "unknown";
}
```

והכותב שאל מה הריפקטורינג המועדף על הקהל, אם בכלל צריך לעשות אחד.

### בואו נסגור את הדיון על ה-code style קודם

על הציוץ הזה [Uncle Bob](http://blog.cleancoder.com/) שלדעתו אם כל המטרה של התוכנה היא לעשות המרה בין מספר למחרוזת הוא היה מעדיף את גרסה מספר 2. 
אני אישית אוהב switch case מאשר if-else (גם כי אני בדר"כ מוסיף סוגריים), אבל בסופו של דבר, המטרה של הקוד (מעבר לזה שהוא אמור לעבוד) זה שהוא יהיה מובן גם בעוד חודש. אז במקרה הספציפי הזה אני הייתי עושה את 2, או את 1 עם return במקום להגדיר משתנה.

### אבל זה הרבה יותר מ-code style

אבל לדעתו (ואני נוטה להסכים איתו פה), שבתוכנה יש עוד שימוש במחזורות על מנת לקבל החלטות לוגיות בקוד.  
וככל הנראה יש עוד מקומות בקוד אשר בטח יש את אותו `if-else-switch` בעוד מקומות, או אפילו הפוך. כלומר שממיר את המחרוזת למספר.

יש שימוש רב ב-`if-else-switch` בקוד תוכנה מודרני. ומה עושים איתו זה בעיה נפוצה.  
הבעיה היא מה קורה כאשר צריך לשנות אותו (וכל קוד עתיד להשתנות)?  
אם הוא קוד משוכפל בהרבה מקומות, אז אנחנו מגדילים את הסיכוי שבשינוי כזה אנחנו נפספס איזה בלוק אחד ונכניס באגים לתוכנה.

אבל יש בעיה גדולה יותר עם `if-else-switch`. זה מבנה התלויות

<figure>

![מבנה התלויות של if-else-switch](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641366376/9b5e437e-acb1-41bd-bcea-bd3ca4dd9524.png)

<figcaption>

מבנה התלויות של if-else-switch

</figcaption>

</figure>

למבנה כזה יש נטייה לכוון מקרים ספציפיים לעבר מודולים ברמה נמוכה יותר.  
כלומר לקוד אשר יש את הבלוק של ה-`if-else-switch` יש תלויות בקוד ברמות הנמוכות יותר.

כמו שכבר הסברתי בסדרה על [design pattern](https://www.thecodeline.org/category/design_pattern/), אנחנו לא אוהבים תלויות בין רמה גבוה לרמה נמוכה.  
זה פוגע בגמישות של הקוד שלנו לשינויים (בקצרה: אם קוד ברמה נמוכה משתנה הוא יכול להשפיע על רמה גבוהה ויגרור עוד שינויים לא רצויים)

אבל הדיאגרמה למעלה מראה לנו בעיה עוד יותר קשה.  
כי לכל תת בלוק ב-`if-else-switch` יש [תלות טרנזיטיבית](https://he.wikipedia.org/wiki/%D7%99%D7%97%D7%A1_%D7%98%D7%A8%D7%A0%D7%96%D7%99%D7%98%D7%99%D7%91%D7%99) בקוד שלנו.  
זה הופך את הבלוק ל_מגנט תלויות_ שנוגע בחלקים גדולים של הקוד מערכת שלנו.  
ובכך הופך את המערכת שלנו למונולית' אחד גדול.

### איך פותרים את זה?

הפתרון לבעיה הזאת היא לפרק את התלויות החיצוניות הללו על ידי שימוש בפולימורפיזם

<figure>

![דיאגרמה לפתרון פולימורפי עבור ה-if-else-switch](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641367278/b166d674-3213-43ab-8e75-7871730c1044.png)

<figcaption>

דיאגרמה לפתרון פולימורפי עבור ה-if-else-switch

</figcaption>

</figure>

בדיאגרמה למעלה אפשר לראות שמודולים ברמה גבוהה יותר משתמשים ב-`base class` שמסתיר תחתיו באמצעות פולימורפיזם קוד ברמה נמוכה יותר.  
הדיאגרמה הזאת היא אותה התנהגות כמו `if-else-switch` רק עם טוויסט קטן.  
ההחלטה באיזה מודול להשתמש צריכה להתבצע לפני שה-`High Level Module` קורא ל-interface של ה-`base class`.

עכשיו נוכל לראות שאין לנו כבר יחס טרנזיטיבי בין ה-`High Level Module` לבין ה-`Low Level`.  
אנחנו בקלות נוכל להחליף את ה-`Low Level`. ואפילו ברמה הרעיונית, אנחנו נוכל לצייר קו על הדיאגרמה הזאת שמחלק בקלות את הקוד לשני החלקים שלו.

עוד נקודת דמיון בין המימוש הפולימורפי לבין `if-else-switch`.  
בשני המקרים אנחנו עושים סוג של אלגוריתם חיפוש על מנת לבצע את העבודה שלהם.  
במקרה של `if else` זה קורה בצורה סינכרונית, ובמקרה של `switch` רוב הקומפיילרים יצרו מעין [lookup table](https://en.wikipedia.org/wiki/Lookup_table).  
במקרה הפולימורפי זה קורה ברמת ההורשה של ה-`base class`.  
ולכן אין יתרון של זיכרון או ביצועים לאף אחד מהשיטות.

### מתי ההחלטה מתבצעת?

<figure>

![שילוב בפולימורפיזם ל-Factory](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641368409/2ede84cb-35c1-4e72-bd14-b6c8105ba8c0.png)

<figcaption>

שילוב בפולימורפיזם ל-Factory

</figcaption>

</figure>

ההחלטה מתבצעת כאשר מופע קונקרטי של ה-`base class` נוצר.  
בתקווה שזה קורה במקום נחמד ובטוח. ובדר"כ אנחנו נשתמש ב-[factory design pattern](https://www.thecodeline.org/the-factory-design-pattern/?swcfpc=1) על מנת לעשות את זה  

בדיאגרמה למעלה אנחנו יכולים לראות שה-`High Level Modules` משתמשים ב-`base class` על מנת לבצע את העבודה שלהם.  
כל חלק בקוד שפעם השתמש ב-`if-else-switch` עכשיו יש לו מתודה ספציפית לקרוא לה ב-`base class`  
כאשר יש business logic אשר קורא למתודה, זה יגיע ל-`Low Level Module` המתאים.  
ה-`Low Level Module` נוצר על ידי ה-`Factory`.  
ה-`High Level` קורא למתודה `make(x)` של ה-`Factory`, כאשר X הוא איזשהו פרמטר המייצג את הפעולה שאנחנו רוצים לבצע (בדומה למספר בדוגמאות מתחילת הפוסט)

רואים את הקו האדום? זה קו גבול שמגדיר את הגבול בין ה-`High Level` לבין ה-`Low Level`.  
רואים שכל התלויות חוצות את הקו הזה?  
זו בדיוק הסיטואציה שאנחנו רוצים שתהיה לנו.

חשוב לשים לב במה אנחנו משתמשים בסוג של X!!  
אם נגדיר אותו כ-`enum`, צריך לשים לב שזה לא דורש הגדרה ב-`High Level` אשר תגרום לחץ הפוך לחצות את הקו האדום.  
אפשר להשתמש במספר או מחרוזת.  
נכון זה לא ייתן לנו בטחון כמו `enum` אבל זה ישמור לנו על ההפרדה (כבר אמרתי שתכנות זה האומנות של סדר עדיפויות?)

### זה נמשך ונמשך

רגע, אבל בעיצוב הנוכחי כל סעיף של ה-`if-else-switch` הופך למתודה, אז כל הוספה של סעיף גורם ל-`base class` להשתנות!  
מעבר לזה שזה יגרום ל-interface הזה להיות ענקי, זה יגרום בערך לכל הקוד להתקמפל על כל שינוי ב-`base class` למרות שלא קרה שום דבר שמשנה אותו (כל מי שעושה import ל-`base class`).

יש הרבה דרכים לפתור את הבעיה הזאת, ואם זה באמת מעניין אתכם אז אני ממליץ לקרוא על _[The Interface Segregation Principle](https://www.thecodeline.org/isp/)_ ועל על _[Acyclic Visitor](https://www.thecodeline.org/acyclic-visitor/?swcfpc=1)_

בכל מקרה, בעיני זה ממש מרתק איך דיון קטן על `if-else-swich` הופך לדיון כל כל מעמיק על ארכיטקטורה.

מקווה שנהניתם, ואני מזמין אתכם להירשם לרשימת תפוצה של הבלוג כדי שלא תפספסו עוד פוסטים כאלה.
