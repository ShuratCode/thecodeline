---
title: "The Decorator Design Pattern"
datePublished: 2022-08-15T00:00:00.000Z
cuid: cmi7ehng8000g02l2bown722h
slug: the-decorator-design-pattern

---


שלום לכולם, הפעם נמשיך במסע שלנו ברחבי ה- [design patterns](#design-patterns).  
אחרי [שפעם קודמת](https://www.thecodeline.org/open-close/) דיברנו על [The open close principle](#open-close), ושם הזכרתי את ה decorator design pattern, אז הגיע הרגע לצלול פנימה

<figure>

https://www.flickr.com/photos/luisplaza/31116970511/in/photolist-PpGBez-BqA7sd-fyCDqU-KnwxVu-RKDpuf-QQCCfw-GR6E59-HrWWkT-2ctj7Cv-J7RB8U-rUrg7o-ERmZsA-2a9GCUu-D4g584-Dz2tqy-SoKUy3-22W2oH1-P6vJGg-MUz2ZX-B626Rt-dGN41m-BJQSFN-2mJZkmw-2kUpntr-R339NY-S8yiM8-MN3X2Q-28pLVcL-2j4V7Aj-XPaEDE-LawQGQ-2hjbc7s-2nSfpzG-2nb844Z-a6TUBp-2kbUUUJ-Mtbe4o-2hvaKKP-RzNdg1-2mPKkqa-UZHGx3-2nq3Wtt-wzDV9A-uG8bnk-GHddmJ-2n82HVJ-xV3NVS-sMmzBs-VUzyQG-C7aiH7

<figcaption>

איך כמו קפה על הבוקר

</figcaption>

</figure>

### הבעיה

אז מעבר לזה ש decorator design pattern עוזר לנו לממש את ה- open close principle, הוא קודם כל בא לענות על בעיה אחרת  
מה קורה כאשר יש לנו אוסף של מחלקות שהן מאוד דומות אחת לשניה אבל עם הבדלים קטנים?  
האופציה הבנאלית היא להוסיף עוד מחלקות בעוד ירושות, מה שיכול לגרום לנו להתפוצצות אוכלוסין בקוד.  
ומה הבעיה בהתפוצצות אולוסין אתם שואלים? הרי אנחנו עובדים ב OOP?  
ובכן לתחזק קוד כזה, זה עינוי שלא ברא השטן.  
האם כל פעם שאנחנו רוצים להוסיף התנהגות חדשה, אנחנו צריכים לקמפל את כל הפרויקט בגלל שהוספנו מחלקה?  
לחפש איפה בדיוק צריך להוסיף, ואת מי בדיוק צריך לרשת?

### ברוך הבא לבית הקפה

דמיינו את בית הקפה האידאלי, שאתם אוהבים לקנות בו קפה.  
זה יכול להיות ארומה, ארקפה או סטארבקס.  
בית הקפה הזה צריך מערכת הזמנות. לקוחות אצלנו יכולים להזמין משקאות מכל מיני סוגים. אבל כל משקה אפשר להכין מכמה סוגים.  
למשל יש לנו קפוצ'ינו, אפשר עם חלב רגיל, חלב דל שומן, עם קפאין או בלי ועוד.  
אז בואו נסתכל רגע על UML, של איך מערכת כזאת אמורה להיות:

<figure>

![גרסה ראשונה של בית הקפה שלנו, כל המימוש הוא על ידי מחלקות](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641290493/4f29dfa8-126c-4ad8-ba99-eb5060847a55.jpeg)

<figcaption>

תרשים UML של בית הקפה שלנו.

</figcaption>

</figure>

המחלקה _**Beverage**_ היא מחלקה אבסטרקטית, כל המשקאות בבית הקפה שלנו יורשות אותה.  
השדה **description** הוא שדה מסוג String, אשר מכיל תיאור מילולי של המשקה. הוא מאותחל בכל מחלקה  
למשל במחלקה **DarkRoast** הוא יהיה: "Most Excellent Dark Roast"  
המתודה **cost()** היא מתודה אבסטרקטית, כלומר היא לא ממומשת במחלקה **Beverage**, כל תת מחלקה (sub class) צריכה להגדיר את המימוש.

בתמונה למעלה יש לנו רק את הסוגי משקאות, אבל כמו שכתבתי, אנחנו יכולים לצרף לכל משקה כזה המון תוספות שונות:

- סוגי חלב שונים
- תוספות מלמעלה כמו: סירופ שוקולד, קרמל כו'

אז בגישה הבאנלית, אנחנו ניצור עבור כל שילוב של משקה כזה מחלקה חדשה.  
כי לכל שילוב כזה יש לנו מחיר אחר.

### למה לא שדות?

בנקודה הזאת אנחנו יכולים לשאול: כל תוספת, וכל סוג חלב יכול להיות שדה בוליאני.  
כי להוסיף חלב שקדים הוא אותו מחיר, לא משנה לאיזה משקה אנחנו מוסיפים אותו.  
כנ"ל גם לתוספות.  
אז המחלקה **Beverage**, תראה ככה:

<figure>

![גרסה שניה של מערכת בית הקפה שלנו.
במקום מחלקות יש לנו שדות](https://www.thecodeline.org/wp-content/uploads/2022/10/decorator_second_uml.jpg)

<figcaption>

גרסה שניה של בית הקפה שלנו, במקום מחלקות אנחנו משתמשים בשדות.

</figcaption>

</figure>

כל השדות החדשים שהוספנו, הם שדות בוליאניים  
בגרסה הזאת, המחלקה האבסטרקטית שלנו **Beverage** ממשמת את המתודה **cost()** בצורה הבאה:

```
public double cost() {
        double condimentCost = 0.0;

        if (hasMilk()) {
            condimentCost += milkCost;
        }

        if (hasSoy()) {
            condimentCost += soyCost;
        }

        if (hasMocha()) {
            condimentCost += mochaCost;
        }

        if (hasWhip()) {
            condimentCost += whipCost;
        }

        return condimentCost;
}
```

  
לדעתי, זה לא הרבה יותר טוב. כי קל מאוד שהמחלקה **Beverage** תהיה ענקית, עם עוד ועוד תוספות, ועוד סוגים של חלב.  
זה לא סקלבילי, ובוודאי שזה לא יהיה קריא.  
ובנוסף אם ניזכר ב [open close principle](#open-close), אז אנחנו רוצים להימנע משינוי של קוד קיים כאשר מתווספת לנו לוגיקה חדשה.

### הכירו את Decorator Design Pattern

אז decorator pattern מוסיף אחרויות נוספת לאובייקט בצורה דינאמית, כלומר בזמן ריצה.  
Decorator מספק אלטרנטיבה לירושה.

אז בואו נסתכל קודם על הUML הרשמי של Decorator pattern, ואז נראה איך אנחנו יכולים להתאים אותו אל הבית קפה שלנו

<figure>

![UML רשמי של Decorator Design Pattern](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641291289/c06111bd-4e9f-45c8-99ab-e3ddf49aae1a.jpeg)

<figcaption>

UML רשמי של Decorator Design Pattern

</figcaption>

</figure>

  
כל **Component** יכול להיות שמיש בזכות עצמו, אבל גם אפשר לעטוף אותו באמצעות Decorator.  
ה **ConcreteComponent** זה האובייקט שאנחנו רוצים להוסיף לו התנהגות חדשה בצורה דינאמית  
לכל **Decorator**, יש קשר של "אחד לאחד" כלומר "עוטף" **Component**. בעצם יש לו שדה של **Component** שעליו אנחנו רוצים להוסיף התנהגות חדשה.  
ה **Decorator** צריך לממש את אותו ה interface או מחלקה אבסטרקטית של ה **Component**

### בית קפה מקושט (Decorator)

אז ככה נראה הUML שלנו עבור בית קפה, אחרי שהחלנו עליו את ה Decorator design pattern:

<figure>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641292034/bcf6a888-71fa-43cb-b5f8-6914773ff14f.jpeg)

<figcaption>

תרשים UML עבור הבית קפה שלנו עם Decorator

</figcaption>

</figure>

אז במקרה שלנו ה **Beverage** משמש כ **Component** שלנו.  
יש לנו 4 מימושים של ה **Beverage**  
והוספנו 4 **Decorator**

אז איך זה עובד בפועל?  
אנחנו ניצור אובייקט ממשי של **Beverage**, נעטוף (Decorate) עם אחד מה Decorator שלנו ובאמצעותו אנחנו נוסיף את המחיר של כל תוספת בצורה דינאמית.  
בואו נראה איך זה נראה בקוד:  
קודם כל הנה המחלקה **Beverage**

```
public abstract class Beverage {

    String description = "Unknown Beverage";

    public String description() {
        return description;
    }
    
    public abstract double cost();
}
```

בסופו של דבר, זו מחלקה די פשוטה.  
  
עכשיו נציג את ה**CondimentDecorator:**

```
package org.decorator.src.final_solution;

public abstract class CondimentDecorator extends Beverage{
    
    Beverage beverage;
    
    public abstract String getDescription();
}
```

נשים לב שהמחלקה הזאת מרחיבה את **Beverage**, ויש לה שדה של אותו מחלקה, כדי שהיא תוכל לעטוף אותו  
בנוסף אנחנו מכריחים שכל ה **Decorator** שלנו יצטרכו לממש את **getDescription**  
  
יאללה, בואו ניצור כמה משקאות:

```
public class Espresso extends Beverage {

    public Espresso() {
        description = "Espresso";
    }

    @Override
    public double cost() {
        return 1.99;
    }
}
```

```
public class Espresso extends Beverage {

    public Espresso() {
        description = "Espresso";
    }

    @Override
    public double cost() {
        return 1.99;
    }
}
```

  
אז פה יש לנו דוגמה של שני משקאות. כל אחד מהם מגדיר את ה **description** שלו בבנאי, ובנוסף, מממש את **cost**  
  
יאללה, אל ה**decorator**

```
public class Mocha extends CondimentDecorator {

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public double cost() {
        return beverage.cost() + 0.2;
    }

    @Override
    public String getDescription() {
        return beverage.description + ", Mocha";
    }
}
```

אז נשים לב שהמחלקה **Mocha** מרחיבה את **CondimentDecorator**  
ובבנאי אנחנו מאתחלים את השדה שאנחנו עוטפים.  
והכי חשוב, גם ב **cost()** וגם ב **getDescription()** אנחנו משתמשים במה שיש ב **beverage** על מנת לחשב את התוצאה הרצויה  
  
אז איך כל זה מתחבר? ככה:

```
ublic class MyCoffeeShop {
    public static void main(String[] args) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() + " $" + beverage.cost());

        Beverage beverage2 = new DarkRoast();
        beverage2 = new Mocha(beverage2);
        beverage2 = new Mocha(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() + " $" + beverage.cost());

        Beverage beverage3 = new HouseBlend();
        beverage3 = new Soy(beverage3);
        beverage3 = new Mocha(beverage3);
        beverage3 = new Whip(beverage3);
        System.out.println(beverage3.getDescription() + " $" + beverage3.cost());
    }
}
```

  
והפלט של זה הוא:

```
Espresso $1.99
Dark Roast Coffee, Mocha, Mocha, Whip $1.99
House Blend Coffee, Soy, Mocha, Whip $1.79
```

### סיכום

היום למדנו בעצם מה זה בכלל Decorator design pattern, למה הוא משמש ואילו בעיות של ירושה הוא פותר לנו.  
מקווה שנהניתם.  
את כל הקוד אתם יכולים למצוא [בgithub הבא](https://github.com/ShuratCode/designPatterns/tree/master/src/main/java/org/decorator)  
  
וכמו תמיד, אשמח לקבל כל שאלה, הערה או הארה שיש לכם
