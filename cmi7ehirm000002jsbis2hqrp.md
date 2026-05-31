---
title: "The Factory Design Pattern"
datePublished: 2022-08-21T00:00:00.000Z
cuid: cmi7ehirm000002jsbis2hqrp
slug: the-factory-design-pattern

---


שלום לכולם, היום אנחנו ממשיכים במסע שלנו ברחבי ה-[design pattern](#design_pattern).  
והפעם אנחנו נדבר על אחד שיוצא לי להשתמש בו די הרבה: The Factory Design Pattern.

יש הרבה דרכים ליצור אובייקטים מעבר לאופרטור new.  
בפוסט הזה אנחנו נלמד שיצירה של אובייקטים היא פעילות שלא תמיד כדאי לעשות אותה ציבורית (public)  
כיוון שלפעמים היא יכולה לגרום לבעיות coupling (מוזמנים להסתכל על הפוסט של [observer](#observer-design-pattern) להסבר)  
ואנחנו לא רוצים בעיות coupling נכון?  
אז בפוסט הזה נלמד ביחד איך Factory Design Pattern יכול להציל אותנו מתלויות מביכות

### כשאתם רואים new תחשבו concrete

כאשר אנחנו משתמשים באופרטור new אנחנו בהכרח מאתחלים concrete class  
כלומר אנחנו כבר עוברים על העקרון של [open close](#open-close)  
ובנוסף, זה יוצר תלות בין ה-class הנוכחית שאנחנו כותבים ל-class שאנחנו יוצרים  
ומכאן שזה מקטין את הגמישות של ה-class שאנחנו כותבים

אם נסתכל על השורה הזאת

```
Duck duck = new MallardDuck();
```

אנחנו רוצים להשתמש באובייקטים אבסטרקטים כמו Duck אבל בפועל אנחנו יוצרים concrete class.  
וכאשר יש לנו אוסף של concrete class יש לנו קוד שנראה ככה

<script src="https://gist.github.com/ShuratCode/60427e90273c03d860e36d274b7bfa22.js"></script>

כאשר אנחנו רואים קוד כזה, אתם יכולים כבר לסמן לעצמכם סימן שאנחנו צריכים לעשות שינויים.  
הסיבה היא שזה קוד שנוטה לגדול עם הזמן, וכנראה זה לא המקום היחידי בו הוא נמצא.  
הוא יופיע בעוד מקומות, מה שמקשה מאוד על תחזוק של הקוד שלנו  
כי כאשר תגיע דרישה להוסיף class חדשה, אנחנו נצטרך לגעת בכמה מקומות שונים.

### אז מה הבעיה עם new

ובכן, טכנית אין בעיה עם new. זה חלק בסיסי בהרבה מאוד שפות מודרניות.  
הבעיה היא עם הקבוע של כל קוד, השינוי  
כאשר אנחנו עוקבים אחרי העקרון של code to interface  
אנחנו מבודדים את הקוד שלנו מהרבה שינויים שיכולים להיות במימושים ספציפיים של מחלקות  
אנחנו מחביאים את השינוי הזה באמצעות polymorphism  
אבל כאשר יש לנו קוד אשר מאתחל הרבה concrete classes  
אנחנו מזמינים לעצמו צרות, כי כאשר יגיע שינוי, אנחנו נצטרך להתאמץ כדי לטפל בו

### מישהו אמר פיצה?

בואו נתחיל לדבר על דוגמה, שתעזור לנו להבין  
אנחנו בעלי פיצרייה, ואנחנו רוצים לכתוב מערכת אשר תעזור לנו לטפל בהזמנות  
אז סיכוי סביר שאנחנו נרצה לכתוב קוד כזה

<script src="https://gist.github.com/ShuratCode/da0bc78c76d4c9a3b7f79df3b0c9c244.js"></script>

אנחנו מעבירים ל-orderPizza את ה-type שאנחנו רוצים ליצור.  
לפי ה-type אנחנו מחליטים איזה concrete class ליצור.  
כל פיצה צריכה לממש את ה-interface Pizza  
אחרי שאנחנו מסיימים ליצור את הפיצה, אנחנו מבצעים סדר פעולות קבוע.

### רק ארבע פיצות?

אבל עם הזמן, אנחנו קולטים שהלקוחות שלנו רוצים יותר פיצות  
אז אנחנו מוסיפים עוד  
ובנוסף כמעט אף אחד לא קונה את הפיצה היוונית שלנו, אז נוריד אותה

<script src="https://gist.github.com/ShuratCode/b78440d09e70bd520268c7f4554f09b8.js"></script>

נשים לב שהחלק בקוד שהשתנה זה הרצף של ה-if-else  
ומה שנשאר קבוע זה הפעולות שאנחנו עושים על הפיצה  
זה צועק לשמיים שאנחנו צריכים לבצע אנקפסולציה על הקוד שמשתנה

כאן נכנס לתמונה ה Factory.  
אנחנו ניצור מחלקה חדשה, שאחראית ליצור את הפיצה שלנו לפי ה-type  
זו כל האחריות שלה. היא תראה ככה

<script src="https://gist.github.com/ShuratCode/7da0dccc132d8d07d9d06a3e7909aa37.js"></script>

מה היתרונות של הגישה הזאת? כל מה שעשינו פה זה לדחוף את הבעיה למקום אחר  
דבר שחשוב לזכור פה הוא של-SimplePizzaFactory יכול להיות הרבה לקוחות  
הרבה מקומות בהם קוראים ל-createPizza  
למשל יכולה להיות לנו מחלקה שמייצגת את התפריט של הפיצריה - PizzaShopMenue - והיא קוראת ל-Factory שלנו על מנת לקבל את הפרטים של הפיצה  
או שיש לנו בכלל שירות משלוחים, שמטפל בפיצות בצורה אחרת  
ולכן בכך שריכזנו את היצירה של הפיצות במקום אחד, אנחנו מרכזית את כל השינויים למקום אחד  
וחוסכים שינויים במקומות אחרים, אלמלא נקטנו בגישה הזאת

אז עכשיו ה-PizzaStore שלנו תראה ככה

<script src="https://gist.github.com/ShuratCode/0666b9376d4c957aa74a8d31aeb30a72.js"></script>

### הגדרה של Factory Design Pattern

מה שהראיתי עד עכשיו נקרא Simple Factory  
הוא לא באמת Design Pattern.  
אבל הוא אבן הבסיס של החלקים הבאים בהמשך.  
הנה ההגדרה שלו

<figure>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641284188/4d8076c1-b2c7-44d8-beb2-8bc10bc8e8b6.jpeg)

<figcaption>

תרשים UML של Simple Factory

</figcaption>

</figure>

ה-PizzaStore הוא ה-client של ה-factory.  
הוא עובר דרך ה-Simple Factory כדי לקבל instance של פיצה  
ה-SimplePizzaFactory, הוא בעצם ה-factory שלנו. שם נוצרים ה-instance של הפיצות  
יש לנו את Pizza, שזה בעצם ה-product של ה-factory.  
במקרה הזה יש לנו מחלקה אבסטרקטית, עם הרבה מחלקות קטנות שיורשות אותה

### הרשת שלנו מתרחבת

הצלחנו כל כך יפה בעבודה של מערכת ההזמנות, שהבעלים החליט להרחיב את הרשת פיצות  
ולפתוח עוד סניף אחד.  
אך פה נכנסת לנו עוד מורכבות, כי במקומות שונים יש פיצות מיוחדות למקום  
אם נרצה להשאר עם SimpleFactory ה-design שלנו יראה ככה:

<figure>

![מימוש של כמה PizzaStore עם Simple Factory](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641285358/dc032435-dc4d-4be7-8b41-2599814ff643.jpeg)

<figcaption>

מימוש של כמה PizzaStore עם Simple Factory

</figcaption>

</figure>

איך זה יראה כקוד?

<script src="https://gist.github.com/ShuratCode/0da2daac580cdb2658b89719d05c2d4c.js"></script>

  
הגישה הזאת עובדת לבינתיים, אבל אז אנחנו מגלים שחלק מהסניפים שלנו מפתחים גישות אחרות להכנת פיצה  
זה פוגע בעסק ובשם שלנו, כי אנחנו רוצים שהתהליך יהיה זהה  
אז בעצם מה שאנחנו רוצים לעשות זה ליצור framework שקושרת בין החנות לבין תהליך הכנת הפיצה ועדיין שהכל יהיה גמיש.

מה שאנחנו הולכים לעשות, זה שאנחנו הולכים להחזיר חזרה את createPizza חזרה לתוך PizzaStore.  
כן מה ששמעתם, עבדנו קשה להוציא ועכשיו מחזירים, תכף אסביר  
הקוד של PizzaStore יראה ככה:

<script src="https://gist.github.com/ShuratCode/c8258610e4ead8dd48c744e5ae292f49.js"></script>

  
ההבדל עכשיו הוא שאנחנו ניתן ל-subclass להחליט איך הפיצה נוצרת  
למשל אם נסתכל על NYStylePizzaStore זה יראה ככה

<script src="https://gist.github.com/ShuratCode/7ea14f3c066aaf106875983610516847.js"></script>

  
אוקי, בו הו, פיצלנו דברים ל-subclass?  
איפה באמת ה-subclass האלו מחליטים משהו? מעבר ל-type?

העניין הוא כשאר אנחנו מסתכלים מהכיוון של ה-PizzaStore  
נזכור שבגרסה הזאת, PizzaStore היא אבסטרקטית.  
וכאשר אנחנו מגיעים לבצע את orderPizza, הקוד כתוב ב-Pizza אבל מי שבפועל מריץ אותו זה אחת ה-subclasses של פיצה  
כאשר orderPizza קוראת ל-createPizza כדי לקבל אובייקט, היא לא יודעת איך הוא נוצר. היא לא יודעת איזו pizza היא מכינה  
מי כן יודעת? ה-subclass כמובן  

### Factory Method

מה שבעצם ראינו פה נקרא Factory Method  
אנחנו בעצם משתמשים ב-method על מנת ליצור factory אשר ייצר לנו את האובייקטים שאנחנו רוצים.  
הקוד המלא של PizzaStore נראה ככה:

<script src="https://gist.github.com/ShuratCode/1bb24c7ade5c5dbbca39002daeed5f4c.js"></script>

  
אנחנו מגדירים את createPizza כabstract, על מנת להכריח כל subclass של PizzaStore  
התבנית של Factory method היא כזאת

```
abstract Product factoryMethod(String type)
```

ה-factoryMethod מחזירה את ה-Product שהיא מייצרת.  
בעצם המתודה הזאת מבודדת את ה-client של ה-factory מהידיעה איזה אובייקט מייצרים  
שימו לב שהפרמטר לא חייב להיות String, יכול להיות כל דבר אשר לפיו מחליטים איזה product לייצר

אוקי בואו נסתכל עכשיו על דוגמה מלאה של factory method

<script src="https://gist.github.com/ShuratCode/85b8031dd9ed9f6b1563f5273a75041a.js"></script>

והפלט נראה ככה

```
--- Making a NY Style Sauce and Cheese Pizza ---
Prepare NY Style Sauce and Cheese Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Grated Reggiano Cheese
Bake for 25 minutes at 350
Cut the pizza into diagonal slices
Place pizza in official PizzaStore box
Ethan ordered a NY Style Sauce and Cheese Pizza

--- Making a Chicago Style Deep Dish Cheese Pizza ---
Prepare Chicago Style Deep Dish Cheese Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Shredded Mozzarella Cheese
Bake for 25 minutes at 350
Cutting the pizza into square slices
Place pizza in official PizzaStore box
Joel ordered a Chicago Style Deep Dish Cheese Pizza

--- Making a NY Style Clam Pizza ---
Prepare NY Style Clam Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Grated Reggiano Cheese
   Fresh Clams from Long Island Sound
Bake for 25 minutes at 350
Cut the pizza into diagonal slices
Place pizza in official PizzaStore box
Ethan ordered a NY Style Clam Pizza

--- Making a Chicago Style Clam Pizza ---
Prepare Chicago Style Clam Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Shredded Mozzarella Cheese
   Frozen Clams from Chesapeake Bay
Bake for 25 minutes at 350
Cutting the pizza into square slices
Place pizza in official PizzaStore box
Joel ordered a Chicago Style Clam Pizza

--- Making a NY Style Pepperoni Pizza ---
Prepare NY Style Pepperoni Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Grated Reggiano Cheese
   Sliced Pepperoni
   Garlic
   Onion
   Mushrooms
   Red Pepper
Bake for 25 minutes at 350
Cut the pizza into diagonal slices
Place pizza in official PizzaStore box
Ethan ordered a NY Style Pepperoni Pizza

--- Making a Chicago Style Pepperoni Pizza ---
Prepare Chicago Style Pepperoni Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Shredded Mozzarella Cheese
   Black Olives
   Spinach
   Eggplant
   Sliced Pepperoni
Bake for 25 minutes at 350
Cutting the pizza into square slices
Place pizza in official PizzaStore box
Joel ordered a Chicago Style Pepperoni Pizza

--- Making a NY Style Veggie Pizza ---
Prepare NY Style Veggie Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Grated Reggiano Cheese
   Garlic
   Onion
   Mushrooms
   Red Pepper
Bake for 25 minutes at 350
Cut the pizza into diagonal slices
Place pizza in official PizzaStore box
Ethan ordered a NY Style Veggie Pizza

--- Making a Chicago Deep Dish Veggie Pizza ---
Prepare Chicago Deep Dish Veggie Pizza
Tossing dough...
Adding sauce...
Adding toppings: 
   Shredded Mozzarella Cheese
   Black Olives
   Spinach
   Eggplant
Bake for 25 minutes at 350
Cutting the pizza into square slices
Place pizza in official PizzaStore box
Joel ordered a Chicago Deep Dish Veggie Pizza
```

  
את הקוד המלא אתם יכולים למצוא [כאן](https://github.com/bethrobson/Head-First-Design-Patterns/tree/master/src/headfirst/designpatterns/factory/pizzas)

### הכרות רשמית עם Factory Method Design Pattern

  
אז אחרי שראינו דוגמה בואו נסתכל על ההגדרה הרשמית.  
יש ל-factory method כמה חלקים עיקריים

- ה-creator: במקרה שלנו זה ה-PizzaStore. לרוב יש בו קוד אשר מבוסס על ה-product האבסטרקטי שאותו הוא יוצר.  
    ה-product נוצר על ידי אחת מה-subclasses של ה-creator.
- ה-product: במקרה שלנו זה כמובן ה-Pizza. גם לו יש subclasses כדי לייצג את הסוגים השונים

  
ה-factory method מגדיר interface ליצירת אובייקטים  
אבל נותן ל-subclasses שלו להחליט איזה class נוצר בדיוק.

<figure>

![UML של factory method](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641286033/b0f7c143-d2e2-40d5-af9e-60b06d347c01.jpeg)

<figcaption>

UML של factory method

</figcaption>

</figure>

### סיכום

כמו תמיד, אשמח לקבל כל הערה, הארה או שאלה שיש לכם  
אשמח אם תפיצו את הפוסט

זהו להפעם. למדנו הרבה היום  
קודם כל מה הבעיה עם new, ולמה לפעמים אנחנו רוצים להחביא את יצירת האובייקטים שלנו  
דיברנו גם על Simple Factory וגם על Factory Method.  
ובסוף ראינו איך הם עוזרים לנו ליצור קוד גמיש יותר וקל להרחבה
