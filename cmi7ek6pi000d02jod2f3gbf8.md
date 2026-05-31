---
title: "Observer Design Pattern"
datePublished: 2022-10-23T00:00:00.000Z
cuid: cmi7ek6pi000d02jod2f3gbf8
slug: observer-design-pattern

---


שלום לכולם,  
היום אנחנו נמשיך במסע שלנו ברחבי ה [design patterns](#design_pattern)  
והפעם יש לי design pattern שדואג שהאובייקטים שלכם ידעו כאשר משהו חשוב להם עומד לקרות.  
קוראים לו Observer Design Pattern.  
הוא אחד ה-design pattern שנמצאים הכי הרבה בשימוש והוא מאוד שימושי.  
בפוסט הזה אנחנו נסתכל על כל מיני אספקטים מעניינים של Observer, למשל קשר של אחד לרבים ו loose coupeling.  
אז יאללה בואו נתחיל

### Subscription, אבל איך?

כולנו מכירים את האופציה להירשם לעדכונים בבלוגים/עתונים/כותבי תוכן (subscription)  
אפילו לבלוג הזה יש (חפשו בתחתית העמוד)  
מה המאפיינים של subscription?

- יוצר תוכן כלשהו מתחיל לפרסם
- אנחנו שרוצים להשאר מעודכנים, נרשמים לרשימת תפוצה שלו
- אפשר לבטל את ההרשמה אם אנחנו כבר לא רוצים לקבל עדכונים
- כל הזמן שיוצר התוכן קיים (מבחינה עסקית כמובן) אנשים ממשיכים להרשם/לבטל הרשמה

אז Observer מתנהג בצורה מאוד מאוד דומה.  
בואו נסתכל על דוגמה כדי להבין את זה יותר טוב

### הכירו את המערכת מזג האוויר שלנו

<figure>

![תרשים זרימה של מערכת מזג האוויר שלנו](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641408346/6c336d79-2918-4d47-b739-0470bb3a2f5e.jpeg)

<figcaption>

תרשים זרימה של מערכת מזג האוויר שלנו

</figcaption>

</figure>

במערכת מזג האוויר שלנו יש כמה מרכיבים  
מצד שמאל אתם יכולים לראות את כל המדדים שלנו,  
כאשר תחנת מזג האוויר מודדת את הנתונים  
יש קטע במערכת שמושך את המידע מתוך התחנה  
והמשימה שלנו היא לכתוב קוד אשר ידע להמיר את המידע הזה לתצוגות שונות באפליקציה:

- מזג אוויר עכשיו
- סטטיסטיקות
- תחזית

בעצם אנחנו מקבלים אובייקט שנקרא WeatherData שאותו אנחנו צריכים לממש.  
זה נראה ככה

<figure>

![אפייון של האובייקט WeatherData](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641409133/d64f375d-e20e-41a7-bf01-a33b295a85cb.jpeg)

<figcaption>

אפייון של האובייקט WeatherData

</figcaption>

</figure>

את כל המתודות של הget אנחנו יכולים להניח שכבר מימשו לנו.  
אנחנו צריכים לממש את measurementsChanged  
אנחנו גם יודעים שהפונקציה הזאת נקראת כל פעם שהמדידות משתנות  
המשימה שלנו היא לבנות את המידע ולעדכן את התצוגות באפליקיציה

אבל בואוו נחשוב על העתיד  
זוכרים על מה דיברתי במבוא ל [design patterns](https://www.thecodeline.org/design-pattern-intro/)?  
הדבר היחיד שקבוע בתוכנה, זה שהיא משתנה  
אז אנחנו רוצים שהתוכנה שלנו תהיה פתוחה להרחבות - 
בעתיד אולי יהיו לנו עוד תצוגות לתמוך בהן. או אולי ימחקו כמה

בואו נסתכל על מימוש ראשוני של הפתרון

<script src="https://gist.github.com/ShuratCode/cc4a51a4293c8356a4e94f85f3705959.js"></script>

קודם כל אנחנו מביאים את כל המידע מחתנת מזג האוויר  
ולאחר מכן אנחנו מעבירים את המידע הלאה לכל התצוגות שלנו.  
אבל אם נסתכל כבר פה, נוכל לראות כמה בעיות

- אנחנו עובדים עם concrete classes ולא עם interfaces
- כאשר תגיע תצוגה חדשה אנחנו נצטרך לשנות את הקוד הקיים שלנו (פוגע בעקרון ה [open close](#open-close))
- אין לנו יכולת להוסיף או להוריד תצוגות בזמן ריצה
- ולא עשינו אנקפסולציה לקוד שמשתנה (פוגע בעיקרון [code to interface](https://www.thecodeline.org/design-pattern-intro/))

אז בסופו של דבר, אנחנו לא פתוחים לשינויים  
וזה, כמו שאתם כבר יודעים יגרור כאבים בעתיד.

### הכירו את observer design pattern

כמו שהסברתי בהקדמה, ה-observer עוזר לנו לדחוף מידע לכל מי שמעוניין  
אם נגדיר אותו רגע בצורה רשמית:

```
Observer design pattern מגדיר יחס של אחד לרבים בין אובייקטים
כך כאשר אחד מהם משתנה כל התלויים שלו מקבלים הודעה על השינוי בצורה אוטומטית
```

אז בעצם יש לנו subject שהוא אחראי לעדכן את כולם  
ויש לנו observer, כלומר צופה, שהוא מעוניין לקבל עדכוני מידע מה-subject.

בואו נסתכל על הUML

<figure>

![UML של Observer design pattern](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641409874/20611853-7f88-4f8f-8cc5-f27bd3ac40a1.jpeg)

<figcaption>

UML של Observer design pattern

</figcaption>

</figure>

נתחיל ב-Subject.  
אובייקטים משתמשים ב-interface הזה על מנת להירשם כ-observers  
וגם כדי למחוק את עצמם מהרשימה

לכל subject יש הרבה observers  
ולכל observer יש מתודה בשם update, שה-subject שלנו צריך לקרוא לך כאשר הוא רוצה להודיע על עדכונים

ה-concreteSubject תמיד מממש את ה-subject interface.  
בנוסף להוספה והסרה של observers, ה-concreteSubject מש את הפונקציה notify  
שבה הוא צריך לקרוא ל-update עבור כל observer  
ל-concreteSubject יכולות להיות עוד מתודות משל עצמו כמובן

ה-concreteObserver יכול להיות כל מחלקה שמממשת את ה-observer interface  
כלומר שיש לה מתודה בשם update.  
כל observer נרשם אצל subject שהוא רוצה לקבל ממנו עדכונים

### הכוח ב-loose coupling

אני רוצה לעצור פה רגע, ולהסביר למה הכוונה ב-loose coupling  
ולמה אנחנו בכלל מדברים על זה בפוסט של observer

כאשר אנחנו מגדירים שני אובייקטים כ-loose coupling, אז הכוונה שלנו היא ששניהם יכולים לבצע אינטרקציה  
אבל הם יודעים מעט מאוד אחד על השני  
וכאשר יש לנו loosely coupling design אז זה נותן לנו גמישות רבה מאוד  
בואו נראה איך design pattern יכול להיות loose coupling:

קודם כל, הדבר היחידי שאפשר לדעת על observer הוא שמממש interface מסויים.  
כל מחלקה שתדבר איתו, לא יודעת מה הוא עושה כאשר קוראים ל-update  
או כל דבר אחר לגביו

אנחנו יכולים להוסיף observers חדשים בכל רגע נתון  
כיוון שה-subject שלנו תלוי רק במתודה update.  
אז אין לנו מניעה להוסיף חדשים, ואנחנו לא צריכים לעדכן את subject או את ה-observers הישנים כאשר אנחנו מוסיפים אחד חדש  
אנחנו יכולים להשתמש ב-observer וב-subject לשימושים אחרים מעבר לעדכון המידע אם יש לנו צורך

ואולי הכי חשוב, שינוי ב-subject וב-observer לא משפיע אחד על השני.  
כל עוד שניהם לא שוברים את ה-interface שהם התחייבו

### שימוש ב observer design pattern באלפיקציה שלנו

אז אחרי שהבנו מה זה בכלל observer design pattern, ומה היתרון שלו  
בואו נשמש בו כדי לעצב את המערכת שלנו מחדש.  
ככה היא תראה

<figure>

![אפליקציית מזג אוויר ממודלת באמצעות observer](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641410509/f8f55f69-8780-4d6d-9da9-4903d23be00a.jpeg)

<figcaption>

אפליקציית מזג אוויר ממודלת באמצעות observer

</figcaption>

</figure>

השראנו את ה-subject interface כמו שהוא היה  
אבל הפעם מי שמממש אותו זה ה-WeatherData. הוא מממש גם את ה-interface, אבל גם מוסיף מתודות משלו  
כל המחלקות אשר מעדכנות את התצוגות של האפליקציה שלנו מממשות שני interfaces  
הראשון כמובן הוא ה-objserver, כדי שהן יוכלו לקבל את המידע  
השני הוא הDisplayElement, כדי שנוכל להשתמש במה שלמדנו במבוא ל-[design pattern](https://www.thecodeline.org/design-pattern-intro/).  
באמצעות ה-interface החדש, אנחנו מאפשרים הרחבה בקלות של המערכת שלנו.

### הלאה אל הקוד

אז בואו נראה איך הכל מתחבר לנו דרך הקוד. קדימה  
נתחיל עם ה-interfaces שלנו

<script src="https://gist.github.com/ShuratCode/619bf3c419e568f4d342e812b83591e0.js"></script>

כאן אין לנו משהו חדש. בדיוק כמו בUML  
רק נפתח פה נקודה קטנה למחשבה.  
האם אתם חושבים שאיך שכתבתי את update היא הדרך הנכונה?  
האם יש דרך יותר נכונה לעשות את זה?  
מה יקרה אם נצטרך להוסיף עוד קריאות? איזה קוד נצטרך לשנות?

  
אז בואו נעבור למשהו יותר מעניין כמו ה-weatherData

<script src="https://gist.github.com/ShuratCode/f103127a4f5cde863434ebec478e5036.js"></script>

כאן הדברים הרבה יותר מעניינים  
כמו שכבר אמרנו, ה-WeatherData הוא המימוש של Subject  
אז הוא מממש את כל המתודות שלו  
שימו לב שהוא משתמש בList כדי לנהל את ה-Observers שהוא צריך לעדכן.  
דבר ראשון שקופץ לעין הוא למה צריך גם את setMeasurements ואגם את measurmentsChanged וגם את notifyObservers  
ובכן, צריך לזכור ששתי הראשונות הן ירושה שקיבלנו ממי שהעביר לנו את כל המערכת.

עכשיו נציג את הdisplays עצמם

<script src="https://gist.github.com/ShuratCode/ebf62dbd42d5e1bad83b5f4e56a63312.js"></script>

הם די strait forward ואין פה לוגיקה מאוד מסובכת  
אני רק אגיד שמאוד קל להוסיף תצוגות חדשות, רק לכתוב class חדש שיממש את שני הinterface ולרשום אותו לweatherData וסגרנו פינה

ועכשיו למה שמחבר הכל ביחד

<script src="https://gist.github.com/ShuratCode/05ec37caaee8089e9c83bb1f3ae1de1b.js"></script>

והפלט שלנו יהיה

```
Current conditions: 80.0F degrees and 65.0% humidity
Avg/Max/Min temperature = 80.0/80.0/80.0
Forecast: Improving weather on the way!
Current conditions: 82.0F degrees and 70.0% humidity
Avg/Max/Min temperature = 81.0/82.0/80.0
Forecast: Watch out for cooler, rainy weather
Current conditions: 78.0F degrees and 90.0% humidity
Avg/Max/Min temperature = 80.0/82.0/78.0
Forecast: More of the same
Current conditions: 62.0F degrees and 90.0% humidity
Avg/Max/Min temperature = 75.5/82.0/62.0

```

### סיכום

זהו להפעם. למדנו הרבה על איך עובד ה-observer design pattern  
ראינו ש-observer design pattern מגדיר יחס של אחד לרבים בין אובייקטים  
ה-subject הוא זה שמעדכן את ה-observers שלו  
כל סוג של אובייקט יכול להיות observer כל עוד יש לו פונקציה של update  
ה-observers הם loosely coupled מה-subject, והבנו גם למה זה מאוד חשוב  
למי שמעוניין, אפשר למצוא את ה-pattern הזה גם ב [Swing](https://www.javatpoint.com/java-swing) או [בRxJava](https://github.com/ReactiveX/RxJava) ובעוד ספריות אחרות

וכמו תמיד, אשמח לקבל כל הערה, הארה או שאלה שיש לכם
