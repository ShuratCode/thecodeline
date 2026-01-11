---
title: "מכאוס לקונטקסט: המדריך הטכני"
seoTitle: "מכאוס לקונטקסט: מדריך טכני קצר"
seoDescription: "מדריך טכני לניהול קוד מתקדם עם Git Worktree ו-AI, כולל דוגמאות מעשיות לשיפור תהליכי עבודה ותיקון באגים"
datePublished: Sun Jan 11 2026 16:13:26 GMT+0000 (Coordinated Universal Time)
cuid: cmk9xo52j000002ih8gpxgbif
slug: cursor-git-worktree-workflow
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1768147853504/a679a245-ae92-4e3f-9927-8602d797a29c.png
tags: cursor-ide, ai-agent, git-worktree

---

# אמ;לק

ביקשתם דוגמאות טכניות? קיבלתם. בפוסט הזה אני חושף את ה״מנוע״ שמתחת למכסה המנוע: מבנה התיקיות שלי, פקודות ה-Git Worktree המדויקות, קבצי ה-Cursor Rule (.mdc) שכתבתי, ואיך דיבאגתי ותיקנתי באג קריטי בפרודקשיין בעזרת שרשרת של סוכני AI.

---

# הקדמה: תראו לי את הקוד

[בפוסט הקודם](https://www.thecodeline.org/mkhvs-lkvntkst), ״מכאוס לקונטקסט״, דיברתי על הפילוסופיה של ניהול עומס קוגניטיבי בעזרת AI. התגובות היו מצוינות, אבל תגובה אחת בלינקדאין תפסה אותי:

> ״וואלה מגניב ביותר! אבל הייתי מצפה מבלוג טכני לכלול גם קצת יותר דוגמאות טכניות…  
> או לפרומפטים או how to worktree ממש מעלה רמה של דברים כאלו

הוא צודק. תיאוריה זה נחמד, אבל בואו נלכלך את הידיים. בפוסט הזה אני אפתח את ה-IDE ואת הטרמינל ואראה לכם צעד-אחר-צעד איך נראה יום בחיי

# שלב 1: זירת המשחקים (The Setup)

כדי להבין את ה-Workflow, צריך להבין את הטופולוגיה הפיזית של הדיסק שלי. אין לי תיקיית פרויקט אחת. יש לי “Workspace” שמאגד את כל העולם שלי:

ככה נראה ה-root שלי:

```bash
$ ll
drwxr-xr-x  common-libraries  # ספריות משותפות
drwxr-xr-x  rnd-documentation # כאן יושבים ה-Software Designs
drwxr-xr-x  first-repo         # מונולית ראשון - Production Branch
drwxr-xr-x  second-repo          # מונולית שני - Production Branch
```

**למה זה קריטי?** תיקיית ה-`first-repo` שאני רואה ב VS COde הראשי שלי היא תמיד ה-Source of Truth. אני לא מפתח עליה ישירות. היא משמשת אותי (ואת ה-AI) למחקר, קריאות קוד, וחיפוש ב-History. היא תמיד נקייה. כנ״ל לגבי `second-repo`

כשאני צריך לתקן באג או לפתח פיצ׳ר, אני לא עושה `git checkout` ומלכלך את הסביבה הזו. אני יוצר עותק זמני וכירורגי.

# שלב 2: הניתוח (Git Worktree)

במקום לעשות `git clone` שלוקח נצח ותופס ג׳יגות, אני משתמש ב-`git worktree`. זו פקודה שיוצרת תיקייה חדשה שמקושרת לאותו בסיס נתונים של גיט, אבל יושבת על Branch נפרד.

הפקודה היא ידנית (אני אוהב שליטה), ופשוטה:

```bash
git worktree add -b bugfix/PROJ-123-fix-bridge ../worktrees/proj-123-bridge master
```

המבנה הכללי:

```bash
git worktree add -b <new branch name> <path to the worktree new dir> <base branch to checkout>
```

**למה זה עדיף על** `git checkout`? כי זה אטומי. יצרתי תיקייה חדשה, עם בראנץ׳ חדש, שמבוסס על המאסטר, בפקודה אחת. התיקייה הראשית שלי `first-repo` נשארה בדיוק באותו מצב שהייתה, ואני יכול לעבור לתיקייה החדשה ולהתחיל לעבוד על ״רצפה נקייה״. ה-Build של התקייה החדשה לא יפריע ל Build בתיקייה הראשית, והקבצים הזמניים לא יתערבבו.

כשאני מסיים לעבוד על ה-worktree אני פשוט מריץ:

```bash
git worktree rm ../worktrees/proj-123-bridge
```

או באופן כללי יותר:

```bash
git worktree rm <worktree directory>
```

**רגע, ומה אם אני עובד על ריפו בודד?** חשוב להבהיר: השיטה הזו לא שמורה רק ל-Monorepos מפלצתיים. גם אם אתם עובדים על פרויקט סטנדרטי לגמרי, `git worktree` הוא שדרוג משמעותי לאיכות החיים. מכירים את זה שאתם באמצע פיצ׳ר מסובך, הכל ״פתוח״ ומבולגן, ופתאום מבקשים מכם לעשות Code Reveiew דחוף או לתקן באג קטן? במקום לעשות `git stash`, להחליף בראנץ׳, לתקן, ואז לחזור ולעשות `git stash pop` - אתם פשוט פותחים Worktree רגעי.

**הבונוס: לא חייבים טרמינל** אם הפקודות למעלה נראות לכם מפחידות, החדשות הטובות ש-Cursor תומך בזה נהדר. כל פעם שאתם פותחים agent חדש אתם יכולים להחליט אם אתם עובדים על אחד מהמצבים הבאים:

* מקומי - local - עבודה על התיקייה הנוכחית
    
* עץ חדש - worktree - קרסר יפתח לכם worktree לבד ואז אפשר להריץ הרבה agents מתוך ה-ui עצמו (אבל זה לא עובד עם כמה ריפו)
    
* בענן - cloud - קרסר ישלח את זה לרוץ בענן, לא התנסיתי בעצמי אבל אשמח לשמוע פידבק מכם.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1768147375438/bc651138-6a78-4165-ae5a-64cc3eedc5f0.png align="center")

# לב 3: המוח (Cursor Rules & Personas)

זה החלק שהרבה ממכם ביקשתם לראות. ה״קסם״ של Cursor הוא לא במודל, אלא ב-**Context** וב-**Persona** שאנחנו מלבישים עליו.

בתיקיית `.cursor/rules` שבתיקייה הראשית - איפה שכל ה repos יושבים - יושבים קבצי ה-`.mdc`. כל קובץ הוא איש צוות וירטואלי עם התמחות ספציפית. הנה הצצה לשני הקריטיים שבהם:

## החוקר (The Researcher)

זה לא סתם ״תסביר לי את הקוד״. זו דמות שעובדת לפי פרוטוקול חקירה נוקשה. שימו לב להנחיה המופרשת בחוק:

```markdown
# Bug Researcher Character
...
## Core Principles
1. Systematic Investigation: Follow precise hierarchy of investigation.
2. Evidence-Based: Base conclusions on code evidence, not assumptions.

## Workflow
1. **Identify**: Extract error message, timestamp, patterns.
2. **Hypothesize**: Generate 3 plausible root causes based ONLY on the error.
3. **Investigate**: Request specific code snippets (Do NOT ask for the whole codebase).
4. **Explain**: Provide a short explanation of WHY the bug is happening.
5. **Wait for Approval**: DO NOT provide fix details until user explicitly approves.
```

(הקובץ המלא ארוך יותר ומכיל תבניות תשובה מדויקות)

## המתכנת (The Coder)

ברגע שהבנו את הבעיה, אנחנו מעבירים את המקל ל״מתכנת״. החוק הזה הוא אובססיבי ל-Clean Code ול-Builds:

```markdown
# Coding Character
...
## Core Principles
- **Test-Driven Development**: Always write tests before implementation.
- **No Comments**: Code must be self-documenting.
- **Build Must Pass**: Never commit if build fails locally.

## Workflow
1. **Analyze Requirements**: Read the Jira ticket or Prompt.
2. **Audit Existing Tests**: Check coverage.
3. **Generate Test Suite**: Happy path, Edge cases.
4. **Implement**: Minimal implementation to pass tests.
5. **Refactor**: Apply strict coding standards (Immutable objects, Constructor injection).
```

# שלב 4: הוכחה מהשטח

תיאוריה זה נחמד, אבל הנה מה שקרה אצלי השבוע. היה לנו באג בפרודקשיין: תהליך שדרוג של רכיב ה-Bridge נתקע ולא סיים.

## שלב החקירה הראשית

פתחתי את Cursor בתיקייה הראשית (דרך ה-ui), טענתי את הלוגים והפעלתי את החוק **Bug Researcher** וזה מה שכתבתי לו:

> @.cursor/rules/characters/bug-researcher-character.mdc I am researching an upgrade to master - but bridge does not finish the upgrade process. In the logs I see..." \[Logs Attached\]

הלוגים הראו הרבה `Failed to handle MapRuleTasks` והודעת אזהרה, אבל שום שגיאה קריטית ברורה. ה-Researcher ניתח את ה-Stack Trace והבין שיש כאן בעיית Recovery. הוא זיהה שפונקציית ה-`finalize()` נכשלת.  
הדיאגנוזה שלו הייתה כירורגית:

> **Root Cause:** Recovery scenario bug. When bridge restarts during upgrade... if upgrade instruction documents are missing... it calls `updateAllStatuses` which throws `NotFoundException`.

## שלב ה״מסירה״

כאן רוב האנשים נופלים. הם מבקשים מה-AI בצ׳אט ״תתקן את זה״. זהבעיה היא שהצ׳אט הוא לא סביבת הפיתוח שלי. במקום זה, ביקשתי מה-Researcher:

> "Write a summarize prompt here in chat for another agent to fix this issue."

הוא ייצר לי **Prompt הנדסי מושלם** שמיועד ל-Agent אחר. הפרומפט הכיל את מיקום הקובץ, מהות התיקון, ומה ההתנהגות המצופה.

## שלב הביצוע

עברתי לטרמינל. עברתי ל-worktree שיצרתי `cd ../worktrees/fix-bridge`), והפעלתי את ה-Agent של Cursor בטרמינל:

```bash
cursor-agent --promp "<prompt from previous step"
```

ה-Agent בטרמינל (שהוא ישות נפרדת מה-IDE הראשי) לקח את הפרומפט, קרא את הקבצים הרלוונטיים, כתב את הטסט, ביצע את התיקון, הריץ קומפילציה ודיווח על הצלחה.

בזמן שהוא עבד בטרמינל? אני עברתי למשימה הבאה ב IDE הראשי.

---

# סיכום: להיות המנצח על התזמורת

השיטה הזאת הופכת אתכם מ״מתכנתים שמקלידים מהר״ ל״מנהלים של צוות AI״.

1. **מפרידים קונטקסט:** תיקייה נקייה למחקר, Worktrees מלוכלכים לעבודה.
    
2. **מפרידים אחריות**: דמות אחת חוקרת ומאבחנת, דמות אחרת כותבת קוד.
    
3. **אוטומציה של התקשורת**: נותנים ל-AI לכתוב את ההוראות ל-AI הבא בתור.
    

זה דורש משמעת בהתחלה (לכתוב את החוקים, לעבוד עם Worktrees), אבל ברגע זה רץ? זה מרגיש כאילו יש לכם כוח על.