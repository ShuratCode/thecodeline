---
title: "לעטוף קריאות - The Command Pattern"
datePublished: 2023-06-04T00:00:00.000Z
cuid: cmi7ekrr2000d02l80159dubo
slug: ltvf-kryvt-the-command-pattern

---


שלום לכולם,  
אנחנו ממשיכים בסדרה של [design-pattern](https://www.thecodeline.org/adapter-design-pattern/), והפעם זה משהו שכמה ביקשו ממני כבר כמה פעמים The Command Pattern.  
בפוסט הזה אנחנו לוקחים את ההכמסה לרמה אחרת לגמרי: אנחנו הולכים לעטוף הפעלה של מתודות.  
היתרון של השיטה הזאת היא שמי שמפעיל את המתודות לא צריך לחשוב יותר מדי איך דברים מתבצעים.

כמו כל הפוסטים האחרים בסדרה הזאת, גם הפוסט הזה מבוסס על הספר הנהדר [Head First Design Pattern](https://www.amazon.com/Head-First-Design-Patterns-Object-Oriented/dp/149207800X).

### חומרה חדשה - שלט לבית חכם

הפעם אנחנו עובדים על משהו חדש, ולא על משחק של ברווזים או מסעדה.  
ביקשו מאיתנו לתכנת שלט לבית חכם.  
השלט שקיבלנו מכיל אופציות לקנפג כמה מכשירים (בפרוטוטייפ יש לנו 8)  
לכל מכשיר יש כפתור של הדלקה (ON) ושל כיבוי (OFF)

במסגרת הפרויקט הזה קיבלנו אוסף של מחלקות עבור כל המכשירים שיודעים להתחבר לשלט החכם:  
`ApplianceControl`, `Stereo`, `CeilingLight`, `OutdoorLight`, `TV`, `FaucetControl`, `CeilingFan`, `Hottub`, `GardenLight`, `GarageDoor`, `Thermostat`, `Sprinkler`  
ועוד רבים וטובים.

כמו שזה נראה, אין סטנדרט אחיד לכל המחלקות הללו. לחלקן יש מתודות בשם `on()`, `off()`, אבל לחלק יש מתודות אחרות  
למשל ל-`GarageDoor` יש מתודות בשם `up()`, `down()`, `stop()`, `lightOn()`, `lightOff()`.  
כלומר אין לנו API אחיד שנוכל להשתמש בו עבור השלט שלנו

### מחשבות ראשונות

אז יש לנו בעצם שלט שהוא יחסית טיפש, הוא לא אמור לחשוב יותר מדי  
אבל כיוון שיש לנו יותר מדי גיוון בכל המכשירים שיכולים להתחבר לשלט זה יוצר לנו בעיה.  
אז איך אנחנו יכולים ליצור תוכנה גנרית? כלומר שהשלט לא ידע יותר מדי איך המכשירים עובדים (על מנת להקטין את התלות)?

### ה-command pattern

כאן נכנס לתמונה ה- `Command Pattern`.  
הוא מאפשר לנו לנתק בין מי שמבקש לבצע פעולה (במקרה שלנו השלט) לבין מי שמבצע אותה (המכשירים החכמים)  
איך אנחנו יכולים לעשות את זה?  
ובכן אנחנו נכנסים אובייקט בין השלט לבין המכשירים שנקרא `command object`.  
האובייקט הזה מבצע הכמסה בין בקשה לביצוע פעולה (למשל להדליק את האור) לבין אובייקט ספציפי (למשל המנורות של הסלון).  
בעצם מה שאנחנו נרצה לעשות הוא לשמור `command object` עבור כל כפתור בשלט.  
השלט לא צריך לדעת איך העבודה הזאת מתבצעת, אלא רק לדעת לאיזו אובייקט לקרוא.

### דוגמה ל-command pattern

זה קצת קשה להבין מה בדיוק הכוונה, אז בואו נסתכל על דוגמה קטנה שתעזור לנו להבין את הנושא.  
הדוגה הזאת כוללת מסעדה, שבה נכנס לקוח ומתיישב  
**הלקוח** שלנו נותן **למלצר** את **ההזמנה** שלו.  
**המלצר** לוקח את **ההזמנה** ומביא אותה לדלפק ומכריז על הזמנה חדשה  
**הטבח** מכין את **ההזמנה**

עכשיו בואו נתבונן יותר לעומק בתהליך הזה:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641435232/ddc5a77a-b5bf-4b1a-bc01-9d7c06c87e9f.png)

- הלקוח יודע מה הוא רוצה, הוא רוצה המבורגר ושייק. הוא רושם את זה בהזמנה שלו ומעביר את זה למלצר

- המלצר לוקח את ההזמנה, וכאשר הוא מוכן הוא קורא למתודה `orderUp()` על מנת להעביר את ההזמנה הלאה

- הטבח עוקב אחר ההוראות של ההזמנה על מנת להכין את הארוחה

אם נסתכל טוב, נוכל לראות שההזמנה היא אובייקט אשר מייצג בקשה להכין ארוחה.  
כמו כל אובייקט אפשר להעביר אותו הלאה.  
יש לו API שמכיל מתודה בודדת `orderUp()`, שמסתיר את הפעולות הנדרשות על מנת להכין את הארוחה.  
בנוסף יש לו רפרנס לאובייקט שאמור לבצע את הפעולה (במקרה שלנו, זה הטבח).  
בעצם ההזמנה מנתקת את התלות בין המלצר לבין כל תהליך הכנת הארוחה.  
המלצר לא צריך לדעת אפילו מה מזמינים, איך מזמינים את זה, ומי בכלל אמור לעשות את זה (הרי הוא רק שם את זה על הדלפק).

למלצר יש עבודה קלה, הוא לוקח את הטופס, ויכול להמשיך לשרת לקוחות נוספים  
עד שהוא מגיע אל הדלפק הזמנות ואז הוא מעביר את ההזמנות שהוא קיבל

הטבח הוא היחידי שיודע איך לבצע את הפעולות בהזמנה, כלומר איך להכין את האוכל.  
הוא זה שממש מממש את המתודות אשר מכינות את האוכל.  
נשים לב שהמלצר והטבח מנותקים לגמרי ולא תלויים אחד בשני - כלומר אפשר להחליף כל אחד מהם וזה לא ישפיע על השני.  
ההפרדה הזאת מתבצעת על ידי ההזמנה.

אם נחשוב על המסעדה שלנו כעל מודל עבור OO design, ראינו איך אפשר להפריד בין אובייקט שמבצע בקשה (לקוח) לבין אובייקט אשר מבצע אותה (טבח).  
במקרה שלנו אנחנו צריכים להפריד בין שלט אשר שמבצע בקשה לבין האובייקט אשר יבצע אותה ממש.  
אנחנו יכולים להתייחס לכל כפתור בשלט בדיוק כמו אל טופס ההזמנה במסעדה, ואז כאשר לוחצים על הכפתור אפשר פשוט לקרוא למתודה המקבילה של `orderUp()` שראינו מהטופס  
ואז האובייקט שצריך לבצע יקח את הבקשה ויבצע אותה.

### ממסעדה ל-design pattern

בואו ננסה לעשות הכללה על הדוגמה של המסעדה שלנו.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641436587/c47518cb-7525-4900-a4a9-bc4b3b24f913.png)

- אנחנו מתחילים ב-`client`. הוא זה שאחראי ליצור את ה-`CommandObject`, והאובייקט הזה מכיל את סט הפעולות שה-`Reciever` צריך לבצע

- ה-`CommandObject` מספק את המתודה `execute()` אשר מנתקת את הסט פעולות

- ה-`client` שלנו קורא ל-`Invoker` עם המתודה `setCommand()` ומעביר לו את ה-`CommandObject` שצריך לבצע.

- בנקודה מסוימת ה-`Invoker` קורא למתודה `execute()` של ה-`CommandObject`

- התוצאה היא ביצוע הפעולות על ידי ה-`Reciever`

### ה-Command Object הראשון שלנו

אז אחרי החפירה הארוכה הזאת בואו נסתכל על קצת קוד  
קודם כל נציג את ה-Command Interface

```
public interface Command {
  public void execute();
}
```

Java

  
עכשיו אנחנו נממש את ה-interface הזה עבור הדלקת אורות

```
public class LightOnCommand implements Command {
  
  Light light;
  
  public LightOnCommand(Light light) {
    this.light = light;
  }
  
  public void execute() {
    light.on();
  }
}
```

Java

  
האובייקט הזה מממש את ה-`Command` interface  
בבנאי אנחנו מעבירים אובייקט ספציפי אשר אמור לבצע את הפעולה, במקרה שלנו אחנו משתמשים ב-`Light`.  
כאשר נקרא ל-`execute` זה האובייקט שנשתמש בו על מנת לבצע את ההוראות.

אז איך אנחנו יכולים להשתמש באובייקט שעכשיו יצרנו?

```
public class SimpleRemoteControl {

  Command slot;
  
  public SimpleRemoteControl() {}
  
  public void setCommand(Command command) {
     slot = command;
  }
  
  public void buttonWasPressed() {
    slot.execute();
  }
}
```

Java

  
בשלט יש לנו מקום אחד (`slot`) לפקודה (`Command`) על מנת לשלוט במכשיר כלשהו.  
יש לנו מתודה אשר מגדירה את ה-`Command` בתוך המקום בשלט, והיא `setCommand()` כמובן.  
אנחנו יכולים לקרוא לה כמה פעמים שנרצה, ובכל פעם אנחנו נחליף את הפקודה אשר הכפתור שלנו יבצע.  
ואנחנו נקרא ל-`slot.execute()` כאשר לוחצים על הכפתור.

### השלט בפעולה

אז אחרי שבנינו את כל האבנים בואו נחבר הכל ונראה איך זה עובד

```
public class RemoteControlTest {
  public static void main(String[] args) {
    SimpleRemoteControl remote = new SimpleRemoteControl();
    Light light = new Light();
    LightOnCommand lightOn = new LightOnCommand();
    
    remote.setCommand(lightOn);
    remote.buttonWasPressed();
  }
}
```

Java

  
ה-`RemoteControlTest` הוא ה-`Client` שלנו עכשיו  
ה-`SimpleRemoteControl` הוא ה-`Invoker`, הוא אחראי להעביר את ה-`Command` על מנת שנוכל להשתמש בו  
לאחר מכן אנחנו יוצרים את ה-`Command` עצמו על מנת להדליק את האור.  
בשורה 7 אנחנו מעבירים את ה-`Command` אל ה-`Invoker` ואז בשורה 8 אנחנו מפעילים את ה-`Command`.

### הגדרה רישמית ל-`Command Design Pattern`

אז ראינו דוגמה לאיך מנתקים אובייקט מבקש מאובייקט מבצע  
ואז ראינו קוד אשר מממש את הדרישות הבסיסיות מהשלט שלנו  
עכשיו אנחנו נגדיר בצורה רשמית את ה-`Command Design Pattern`

ה-`Command Design Pattern` מכמיס בקשה לביצוע פעולה כאובייקט, ובכך מאפשר לנו לבצע העדפה, לשמור בלוג או בתור כמה פעולות.

בואו נפרק את זה,  
אנחנו יודעים שה-`command` מכמיס את הבקשות על ידי חיבור לסט של פעולות עבור `receiver` יחיד.  
הדרך בה השגנו את זה היא על ידי חשיפה של מתודה בודדת `execute()`.  
וכאשר קוראים ל-`execute()` אנחנו בעצם מפעילים את סט הפעולות המדובר.  
מבחוץ אף אחד לא יודע איך סט הפעולות הזה מתבצע ועם איזה `receiver`.  
גם ראינו איך לשנות או לקבוע התנהגות של אובייקטים באמצעות `command`, מה שלא ראינו עדיין זה איך שומרים בקשות על מנת שנוכל לבצע Undo.  
אנחנו נגיע לזה בעוד רגע, אבל בואו קודם נגדיר לנו את ה UML

ה-`Client` אחראי ליצור את ה-`ConcreteCommand` ולהגדיר לו את ה-`Receiver`.  
ה-`Invoker` מחזיק את ה-`command` ובזמן מסויים מבקשת ממנו לבצע את הפעולות שלו על ידי קריאה למתודה `execute()`  
`Command` מגדיר interface עבור כל ה-`Commands`. שימו לב שהוספנו עכשיו גם את `undo()`  
ה-`Receiver` יודע איך לבצע את הפעולות. כל מחלקה יכולה להיות `Receiver`.  
ה-`ConcreteCommand` מגדירה קשר בין פעולה לבין `Receiver`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763641437402/d157ea2d-b283-4197-8a6c-30c394aacad5.png)

### הקצאה של פקודות לכפתורים

אז אחרי שהבנו לעומק מהו ה-`Command Design Pattern`, אנחנו יכולים לממש את השלט שלנו.  
אנחנו הולכים להקצות לכל כפתור `Command`. זה הופך את השלט שלנו ל-`Invoker`, כי הפעולות ירוצו רק כאשר מישהו ילחץ על הכפתור.

```
public class RemoteControl {
  Command[] onCommands;
  Command[] offCommands;
  
  public RemoteControl() { 
    onCommands = new Command[7];
    offCommands = new Command[7];
    
    Command onCommand = new NoCommand();
    for (int i = 0; i < 7; i++) {
      onCommands[i] = noCommand;
      offCommands[i] = noCommand;
    }
  }
  
  public void setCommand(int slot, Command onCommand, Command offCommand) {
    onCommands[slot] = onCommand;
    offCommands[slot] = offCommands;
  }
  
  public void onButtonWasPushed(int slot) {
    onCommands[slot].execute();
  }
  
  public void offButtonWasPushed(int slot) {
    offCommands[slot].execute();
  }
}
```

Java

  
הקוד הזה הוא די פשוט (אם יש שאלות אל תפחדו לכתוב בתגובות)  
עכשיו בוא נסתכל על ה-`Command` עצמו

```
public class LightOffCommand implements Command {
  Light light;
  
  public LightOffCommand(Light light) {
    this.light = light;
  }
  
  public void execute() {
    light.off();
  }
}
```

Java

  
כמו שאתם יכולים לראות, אנחנו מעבירים ל-`LightOffCommand` את האובייקט `Light` שהוא בעצם ה-`Receiver` שלנו.  
וכאשר אנחנו קוראים ל-`execute()` ה-`Light` מבצע את העבודה של לכבות את האור.  
בואו נסתכל על עוד דוגמה

```
public class StereoWithCDCommand implements Command {
  Stereo stereo;
  
  public StereoOnWithCDCommand(Stereo stereo) {
    this.stereo = stereo;
  }
  
  public void execute() {
    stereo.on();
    stereo.setCD();
    stereo.setVolume(11);
  }
}
```

Java

  
בדוגמה הזאת אנחנו מגידירים את הפקודה להדליק את הסטריאו עם CD.  
נשים לב שאנחנו מקבלים `Stereo` בתור ה-`Receiver` שלנו.  
ובתוך `execute()` אנחנו מבצעים שלוש פעולות שונות על מנת להשלים את הפקודה.

### מה עם ה-Undo()?

אז קודם כל עלינו לעדכן את ה-interface של `Command`

```
public interface Command {
  public void execute();
  public void undo();
}
```

Java

  
ועכשיו נוכל לעדכן את `LightOnCommand` בהתאם

```
public class LightOffCommand implements Command {
  Light light;
  
  public LightOffCommand(Light light) {
    this.light = light;
  }
  
  public void execute() {
    light.off();
  }
  
  public void undo() {
    light.off();
  }
}
```

Java

  
אבל איך נוכל לתמוך בזה ברמת השלט?  
ובכן אנחנו נצטרך כפתור פיזי שייצג את הקריאה לפעולת ה-Undo  
ולאכן מכן, אנחנו נצטרך לבצע עדכון של השלט על מנת לתמוך בזה

```
public class RemoteControl {
  Command[] onCommands;
  Command[] offCommands;
  Command undoCommand;
  
  public RemoteControl() { 
    onCommands = new Command[7];
    offCommands = new Command[7];
    
    Command onCommand = new NoCommand();
    for (int i = 0; i < 7; i++) {
      onCommands[i] = noCommand;
      offCommands[i] = noCommand;
    }
    undoCommand = noCommand;
  }
  
  public void setCommand(int slot, Command onCommand, Command offCommand) {
    onCommands[slot] = onCommand;
    offCommands[slot] = offCommands;
  }
  
  public void onButtonWasPushed(int slot) {
    onCommands[slot].execute();
    undoCommand = onCommands[slot];
  }
  
  public void offButtonWasPushed(int slot) {
    offCommands[slot].execute();
    undoCommand = offCommands[slot];
  }
  
  public void undoButtonWasPushed() {
    undoCommand.undo();
  }
}
```

Java

  
בצורה הזאת כל פעם שנלחץ כפתור בשלט, אנחנו שומרים את זה כפעולה האחרונה שבוצעה  
וכך כאשר ילחצו על פעולת ה-undo אנחנו פשוט נקרא ל-undo המתאים.

### סיכום

אז זהו הסיפור של `Command Design Pattern`.  
בפוסט הזה למדנו איך שימוש ב- `Command` מנתק את האובייקט שמבקש את הבקשה לבין האובייקט שמבצע אותה.  
יש לנו את ה-`Invoker` שהוא מבצע בקשה לביצוע `Command` על ידי קריאה לפונקציה `execute()`

אתם יכולים לראות את הקוד המלא [כאן](https://github.com/bethrobson/Head-First-Design-Patterns/tree/master/src/headfirst/designpatterns/command).

כמו תמיד, אשמח לקבל כל שאלה, הערה או הארה שיש לכם.
