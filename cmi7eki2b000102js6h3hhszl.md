---
title: "להיות סתגלתנים חלק ב' - פאסד Facade"
datePublished: 2023-04-02T00:00:00.000Z
cuid: cmi7eki2b000102js6h3hhszl
slug: lhyvt-stgltnym-hlk-v-fsd-facade

---


שלום לכולם,  
היום אנחנו נמשיך במסע שלנו ברחבי ה-[design patterns](https://www.thecodeline.org/category/design_pattern/).  
בפעם הקודמת דיברנו על [Adapter Design Pattern](https://www.thecodeline.org/adapter-design-pattern/).  
והיום אנחנו נדבר על design pattern דומה שקוראים לו `Facade`

כמו כל הפוסטים בסדר הזאת, גם הפוסט הזה מבוסס על הספר הנהדר [Head First Design Pattern](https://www.amazon.com/Head-First-Design-Patterns-Object-Oriented/dp/149207800X)

כשדיברנו על ה-`Adapter` ראינו איך הוא מסוגל להמיר interface אחד ל-interface אחר.  
השגנו את זה בג'אווה בכך שהעברנו ל-`Adapter` את ה-interface שאנחנו רוצים להמיר, וה-`Adapter` ממימש את ה-interface שאליו אנחנו רוצים להגיע.

גם ה-`Facade` משנה interface, אבל מסיבה שונה.  
הוא מנסה לפשט את ה-interface.  
הוא מסתיר את כל המורכבות של interface אחד או יותר מאחורי interface נקי ומסודר

### הקולנוע הביתי

לפי שנצלול לפרטים של ה-Facade, נסתכל קודם על המערכת שלו.  
הפעם אנחנו לא מדברים על ברווזים (איזה מזל הא?)  
אלא הפעם אנחנו נרכיב לנו מערכת קולנוע ביתית כדי שנוכל להתפנק בבית ולראות סרטים באיכות גבוה

ככה נראית המערכת שלנו

זוהי מערכת מאוד מורכבת, כיוון שיש לה הרבה חלקים שמשתתפים.

<figure>

![מבנה המערכת של הקולנוע הביתי שלנו](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641423730/8db36c5b-537f-4cbf-9bc4-f4bf33b7b0e8.png)

<figcaption>

מבנה המערכת של הקולנוע הביתי שלנו

</figcaption>

</figure>

### אז איך רואים סרט (הדרך הקשה)

אז איך באמת אנחנו עובדים עם המערכת המורכבת הזאת?  
והכן הנה רשימת הצעדים שצריך לעשות

1. להדליק את מכונת הפופקורן

3. להכניס פופקורן להכנה

5. לעמעם את האורות

7. להוריד את המסך

9. להדליק את המקרן

11. לסמן למקרן שה-input שלו הוא ה-streaming player

13. לשנות את המצב של המקרן למסך רחב

15. להדליק את המגבר

17. לסמן למגבר שה-input שלו הוא ה-streaming player

19. להעביר את המגבר למצב surround sound

21. לכוון את עוצמת המגבר לבינונית (5)

23. להדליק את נגן הסטרימינג

25. להתחיל לנגן סרט

וואלה כואב הראש רק מלחשוב על כל הבאלגן הזה.  
רק כדי לראות סרט הייתי צריך לקחת חצי שעה של הכנה.  
אבל איך כל הפעולות הללו נראות מבחינת קוד?

לא יודע מה איתכם, אבל לדעתי שימוש ב6 קלאסים שונים בשביל לראות סרט בודד זה הרבה יותר מדי.  
אבל זה לא יגמר פה, מה קורה כשסרט נגמר? איך אנחנו מכבים את הכל? אנחנו נצטרך להשתמש ב-6 הקלאסים הללו רק בסדר הפוך!  
ומה יקרה אם נרצה לשדרג את המערכת שלנו, אנחנו נצטרך ללמוד תהליך אחר לגמרי.  
לא נחמד

Java

```
// Prepare popcorn
popper.on();
popper.pop();

// Dim the light to 10%
lights.dim();

// Put the screen down
screen.down();

// Turn on the projector5 and put it in widescreen mode for the movie
projector.on();
projector.setInput(player);
projector.wideScreenMode();

// Turn on the amp, set it to the streaming player, put it in surround-sound mode, and set the volume to 5;
amp.on();
amp.setStreamingPlayer(player);
amp.setSurroundSound();
amp.setVolume(5);

// Turn on the Streaming player and FINALY watch the movie
player.on();
player.play(movie);
```

### אורות, מצלמה, פאסאד!

`Facade` הוא בדיוק מה שאנחנו צריכים כאן: עם `Facade` אנחנו יכולים לקחת מערכות מורכבות ולהפוך אותן קלות לשימוש על ידי מימוש ה-`Facade Interface`.  
ה- `Facade Interface` בעצם חושף לנו interface אשר הוא קל לשימוש, והוא מחביא את כל המורכבות של המערכת מאחוריו, מבלי שאנחנו נצטרך להתעסק איתה.  
אז איך `Facade` פועל? ובכן:

אז בואו ניצור את ה-`Facade` של המערכת לקולנוע ביתית שלנו. על מנת לעשות את זה, אנחנו צריכים ליצור מחלקה חדשה בשם `HomeTheaterFacade`, אשר חושפת מתודות פשוטות לשימוש, כמו `watchMovie()`.

ה-`Facade` מתייחסת לכל המערכת הקולנוע הביתי כאל תת מערכת, וקלאס הזה קורא למרכיבים של התת מערכת על מנת לממש את `watchMovie()`.

הלקוח של ה-`Facade` יכול עכשיו לקרוא למתודות בצורה נוחה על מנת להשיג את מה שהוא רוצה, והוא לא צריך להתעסק עם תת המערכת או בכלל להכיר איך היא בנויה.

ה-`Facade` ישאיר את תת המערכת **גלויה** ונגישה לשימוש מבחוץ.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641424601/dca380c9-402c-48c1-bdb4-0111a3568ed7.png)

חשוב לציין שה-`Facade` לא מבצע הכמסה לתת מערכת. הוא רק מפשט את ה-interface.  
ויתרון נוסף שיש ל-`Facade` מעבר לפשטות, שהוא מאפשר לנתק בין הקליינט לבין התת מערכת, ובכך מקל על שינויים בעתיד

אז איך נראה הקוד?

קודם כל אנחנו יכולים לראות שאנחנו מגדירים את כל חלקי התת מערכת כשדות של ה-`Facade`.  
ה-`Facade` מקבל את כל החלקים הללו כפרמטרים בבנאי.

`watchMovier()` עוקב אחרי אותו רצף פעולות שהיה לנו לפני זה, אבל עוטף את הכל ומסתיר את רצף הפעולות. נשים לב שעבור כל פעולה אנחנו משתמשים בחלק של התת מערכת

וה-`endMovie()` מכבה את כל חלקי התת מערכת.

Java

```
public class HomeTheaterFacade {
  Amplifier amp;
  Tuner tuner;
  StreamingPlayer player;
  Projector projector;
  TheaterLights lights;
  Screen screen;
  PoppcornPopper popper;
  
  public HomeTheaterFacade(Amplifier amp,
        Tuner tuner,
        StreamingPlayer player,
        Projector projector,
        Screen screen,
        TheaterLights lights,
        PopcornPopper popper) {
  
    this.amp = amp;
    this.tuner = tuner;
    this.player = player.
    this.projector = projector;
    this.screen = screen;
    this.lights = lights;
    this.popper = popper;      
  }
  
  public void watchMovie(String movie) {
    System.out.println("Getr ready to watch a movie...");
    popper.on();
    popper.pop();
    lights.dim(10);
    screen.down();
    projector.on();
    projector.wideScreenMode();
    amp.on();
    amp.setStreamingPlayer(player);
    amp.setSurroundSound();
    amp.setVolume(5);
    player.on();
    player.play(movie);
  }
  
  public void endMovie() {
    System.out.println("Sutting movie theater down...");
    popper.off();
    lights.on();
    screen.up();
    projector.off();
    amp.off();
    player.stop();
    player.off();
  }
}
```

### הזמן לראות סרט בדרך הקלה

אז בואו נחבר את הכל עכשיו

בשלב הראשון אנחנו יוצרים את כל החלקים של התת מערכת. בדרך כלל קליינט מקבל `Facade` מוכן כבר והוא לא צריך לבנות הכל בעצמו  
לאחר מכן אנחנו נאתחל את ה-`Facade` עם כל חלקי התת מערכת.  
ובסוף אנחנו משתמשים ב-interface הפשוט שלו על מנת לראות סרט.

Java

```
public class HomeTheaterTestDrive {
  public static void main(String[] args) {
    // initiate components here
    
    HomeTheaterFacade homeTheater =
      new HomeTheaterFacade(amp, tuner, player,
                projector, screen, lights, popper);
    homeTheater.watchMovie("SHAZAM");
    homeTheater.endMovie();
  }
}
```

### הגדרה של `Facade Pattern`

על מנת להשתמש ב- `Facade Patter` אנחנו יצרנו מחלקה אשר מפשטת interface של תת מערכת מורכבת.  
בניגוד ל-`design patterns` אחרים, ה-`Facade` הוא די פשוט. אבל עדיין יש פה אבסטרקציה

ה-`Facade Design Pattern` מספק לנו interface מאוחד עבור אוסף של interfaces בתת מערכת.
ה-`Facade` מגדיר interface ברמה גבוהה יותר ומפשט את השימוש בתת מערכת

<figure>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641425299/c3e0c451-4127-415b-8994-1195f1c9a2aa.png)

<figcaption>

תרשים UML עבור `Facade`

</figcaption>

</figure>

### עיקרון האי ידיעה - The Principle of Least Knowledge

עקרון האי ידיעה מנחה אותנו להקטין את האינטרקציות בין אובייקטים לכמה "חברים" קרובים.  
ההגדרה שלו היא:

```
דבר רק עם החברים הקרובים ביותר שלך
```

אבל מה זה אומר בכלל?  
זה אומר שכאשר אנחנו מעצבים מערכת, אנחנו צריכים להזהר עם כמות האינטרקציות שכל אובייקט שלנו מבצע.  
העיקרון שומר עלינו מלעצב מערכות אשר הרבה מחלקות משולבות אחת בשניה.

#### איך לא לזכות בחברים חדשים והשפעה על אובייקטים

אוקי, אז איך אנחנו עושים את זה?  
העיקרון של אי הידיעה מספק לנו כמה כללי אצבע:  
בכל מתודה של כל אובייקט, הוא יכול לקרוא למתודות של:

- האובייקט עצמו

- אובייקטים אשר עברו כפרמטרים למתודה עצמה

- כל אובייקט שהמתודה יצרה בעצמה

- כל component של האובייקט (יחס של Has-A)

נשמע מאוד נוקשה?  
מה הבעיה לקרוא למתודות של אובייקטים שקיבלנו ממתודות אחרות?  
הבעיה היא שאנחנו קוראים לחלק של אובייקט אחר, ובכך אנחנו מגדילים את התלות של המחלקה הנוכחית בעוד מחלקה אחרת  
בואו נתסכל על דוגמה:

Java

```
public float getTemp() {
  Thermometer thermometer = station.getThermometer();
  return thermometer.getTemperature();
}
```

בדוגמה הזאת, אנחנו מקבלים את האובייקט `Thermometer` ממתודה של `station` ואז אנחנו קוראים למתודה של `Thermometer`  
אז לא רק שאנחנו תלויים ב-`station` עכשיו אנחנו גם תלויים ב-`Thermometer`.  
הגדלנו את התלויות שלנו וסיבכנו את המערכת, כי אם `Thermometer` ישתנה, אנחנו נצטרך לשנות קוד גם כאן.  

Java

```
public float getTemp() {
  station.getTemperature();
}
```

בדוגמה הזאת אנחנו משאירים את התלות שלנו רק ב-`station`.  
הוא זה שידאג לנו לקבלת הטמפרטורה.

עכשיו בואו נסתכל על דוגמה אשר תסביר איך אפשר להפר כל חוק של עיקרון אי הידיעה

Java

```
public class Car {
  Engine engine; // engine is a component of Car
  
  public Car() {
    // initialize engine
  }
  
  public void start(Key key) {
    Doors doors = new Doors();
    boolean authorized = key.turns(); // We can call to key methods because we got it as parameter
    if (authorized) {
      engine.start(); // We can call to start() because engine is a component
      updateDashboardDisplay(); // Car method so we can call it
      doors.lock(); // the method start() created doors so it can call its methods
    }  
    
    public void updateDashboardDisplay() {
      //update display
    }
  }
}
```

### פאסאד ועיקרון אי הידיעה

ב-`Facade` הקליינט לא צריך להכיר את כל חלקי התת מערכת.  
הוא לא צריך להיות תלוי בהם  
הוא מכיר רק את ה-`Facade` ותלוי רק בו  
ובכך אנחנו שומרים על עיקרון אי הידיעה, ואם התת מערכת משתנה, הקליינט שלנו לא צריך להשתנות בעקבות זה.

### סיכום

בפוסט הזה למדנו שכאשר אנחנו צריכים לפשט אוסף של interfcaes בתת מערכת מורכבת, אנחנו יכולים לעטוף את הכל ב-`Facade` ולחשוף interface פשוט.  
ה-`Facade` מנתק את הקליינט מהתת מערכת, ובכך שומר עליו משינויים בה

מקווה שנהניתים, מוזמים להרשם לרשימת תפוצה בתחתית העמוד על מנת להשאר מעודכנים  
וכמו תמיד, אני אשמח לקבל כל הערה, הארה או שאלה שיש לכם
