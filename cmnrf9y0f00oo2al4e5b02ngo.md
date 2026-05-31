---
title: "ניהול רענון דאטה ב-Tableau Cloud עם Airflow"
seoTitle: "ניהול רענוני Tableau ב-Airflow: פתרון בעיות קונקירנסי ו-PAT"
seoDescription: "למדו איך להתגבר על שגיאת 401002 ב-Tableau Cloud ואיך לנהל רענוני דאטה במקביל באמצעות Airflow Pools ו-JWT בתוך סביבת MWAA."
datePublished: 2026-04-09T11:57:28.817Z
cuid: cmnrf9y0f00oo2al4e5b02ngo
slug: tableau-cloud-airflow
cover: https://cdn.hashnode.com/uploads/covers/691f0493feafad720fa018a5/58a7bb16-4314-47d6-8367-fe457f7e49a3.jpg
tags: airflow, tableau, dataengineering, manage-workflow-airflow

---

# אמ;לק

כאשר מריצים רענוני Extract ב-Tableau Cloud מתוך Airflow (במיוחד ב-MWAA), שימוש ב-Personal Access Tokens (PAT) יוצר צוואר בקבוק של סשן יחיד (Linear Session). ניסיון לעבודה מקבילית גורם לביטול סשניים (Error 401002). הפתרון לטווח הקצר הוא שימוש ב-Airflow Pools לסריאליזציה של הקריאות ל-API, בעוד הפתרון לטווח ארוך הוא מעבר ל-JWT Connected Apps, בכפוך למגבלות גרסאות ה-Providers ב-MWAA

* * *

# רקע: השאיפה למקביליות ב-BI Orchestration

בסביבת הפרודקשיין שלנו, המבוססת על **AWS Managed Workflows for Apache Airflow (MWAA)** בגרסה 2.10.3, המטרה הייתה פשוטה: לנהל עשרות זרימות BI עצמאיות במחזור של 4 שעות.

הארכיטקטורה נראתה מצוין: DAG מבוסס קונפיגורציה, שבו כל Flow כולל טרנספורמציה ב-dbt cloud ולאחריה רענון של ה-Datasource ב-Tableau. רצינו מקביליות מקסימלית כדי שזמן הריצה הכולל ייקבע לפי ה-Flow הארוך ביותר, ולא לפי סכום כל הריצות. אבל אז פגשנו את ה-API של Tableau.

# איך זה עובד (או למה ה-PAT שובר אתכם)

הבעיה הראשונה נעוצה בניהול הסשנים של **Tableau REST API**. כשאנו משתמשים ב-PAT, ה-Site logic אוכף קשר לינארי קשיח:

1.  **טאסק ראשון** מבצע `sign_in()` ומקבל **טוקן-1**
    
2.  **טאסק שני** (רץ במקביל) מבצע `sign_in()` עם אותו PAT ומקבל **טוקן-2**
    
3.  הזמן הזה **Tableau Cloud** מבטל מיד את **טוקן-1**
    
4.  **טאסק ראשון** מנסה להשתמש בטוקן המבוטל ומקבל `(Unauthorized Access) 401002`
    

זהו מנגנון אבטחה מתועד, אך הוא כופה סריאליזציה מלאה על כל מה שמנסה להתחבר תחת אותה זהות PAT.

![](https://cdn.hashnode.com/uploads/covers/691f0493feafad720fa018a5/6987bbb8-3609-4097-91d5-6d4958417627.jpg align="center")

* * *

# ניתוח האפשרויות: ממבוי סתום לפתרון

ניסינו לחזור לשיטה הישנה של שם משתמש וסיסמא, אך נתקלנו ב-**MFA Wall**. מאז 2022, Tableau מחייבת MFA, מה שהופך אוטומציה מבוססת סיסמה לבלתי אפשרית ללא התערבות אנושית או מעקפים מורבים.

הפתרון ה״נכון״ הוא **JWT Connected App**, המאפשר Stateless Authentication ותומך בסשנים מרובים במקביל. עם זאת, ב-MWAA גילינו את ״שומר הסף של הסביבה״: גרסת ה-Tableau Provider נעולה על 4.5.0, בעוד שתמיכה ב-JWT נוספה רק ב-5.2.0. ניסיון לדרוס את הגרסה ב`requirements.txt` נכשל בגלל קובץ ה-Constraints הקשיח של AWS.

**טבלת השוואת אסטרטגיות אימות:**

<table style="min-width: 125px;"><colgroup><col style="min-width: 25px;"><col style="min-width: 25px;"><col style="min-width: 25px;"><col style="min-width: 25px;"><col style="min-width: 25px;"></colgroup><tbody><tr><td colspan="1" rowspan="1"><p><strong>שיטת אימות</strong></p></td><td colspan="1" rowspan="1"><p><strong>מודל קונקירנסי</strong></p></td><td colspan="1" rowspan="1"><p><strong>תואם MFA?</strong></p></td><td colspan="1" rowspan="1"><p><strong>תמיכה ב-MWAA (Native)</strong></p></td><td colspan="1" rowspan="1"><p><strong>סטטוס</strong></p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>PAT</strong></p></td><td colspan="1" rowspan="1"><p>ליניארי / סשן יחיד</p></td><td colspan="1" rowspan="1"><p>כן</p></td><td colspan="1" rowspan="1"><p>נתמך (4.5.0)</p></td><td colspan="1" rowspan="1"><p>בשימוש (עם Pool)</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>User/Pass</strong></p></td><td colspan="1" rowspan="1"><p>רב-סשנים</p></td><td colspan="1" rowspan="1"><p><strong>לא</strong></p></td><td colspan="1" rowspan="1"><p>נתמך</p></td><td colspan="1" rowspan="1"><p>חסום ע"י MFA</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Connected App (JWT)</strong></p></td><td colspan="1" rowspan="1"><p>רב-סשנים / Stateless</p></td><td colspan="1" rowspan="1"><p>כן</p></td><td colspan="1" rowspan="1"><p>מגרסה 5.2.0 בלבד</p></td><td colspan="1" rowspan="1"><p>Roadmap</p></td></tr></tbody></table>

* * *

# הפתרון המעשי: Airflow Pools כשסתום לחץ

התובנה הייתה שלא צריך לשנות את הטופולוגיה של ה-DAG, אלא להשתמש במנגנון ניהול המשאבים של Airflow כדי לנהל את צוואר הבקבוק.

יצרנו **Airflow Pool** בשם `tableau_refresh_pool` עם Slot אחד בלבד. בעוד שמשימות ה-dbt רצות במקביל ומנצלות את כל ה-Workers, משימות הטאבלו ממתינות בתור. כך אנחנו מבטיחים שרק Worker אחד מחובר ל-Tableau בכל רגע נתון, מה שמונע את ה-Race Condition.

## דוגמת קוד: DAG Factory מבוסס קונפיגורציה

כך אנחנו מחילים את המגבלה בצורה דינמית:

```python
# Deploy TableauOperator with Pool
for luid in conf['tableau_datasource_luids']:
    TableauOperator(
        task_id=f"refresh_{luid[:8]}",
        site_id="my_site",
        resource="datasources",
        method="refresh",
        find=luid,
        pool="tableau_refresh_pool" # הקסם שמונע 401002
    ) >> dbt_task
        
```

* * *

# סיכום

1.  **Auth is Architecture**: שיטת האימות היא לא רק עניין של אבטחה, היא מכתיבה את יכולת ה-Scaling של המערכת.
    
2.  **מגבלות נסתרות:** בשירותים מנוהלים כמו MWAA, תמיד תבדקו את קובץ ה`constraints.txt` לפני שאתם מתכננים פיצ׳רים שמסתמכים על גרסאות provider חדשות.
    
3.  **Pools הם כלי אסטרטגי:** השתמשו בהם כדי להפריד בין הטופולוגיה הלוגית של ה-DAG לבין מגבלות פיזיות של API חיצוניים.