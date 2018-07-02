---
layout: post
title: Testing Angular - Overview 
description: "בפרק הראשון בסדרה, נדבר על הקדמה לבדיקות"
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [Testing]
image: '/images/posts/testingAngularOverview.jpg'

---



### הקדמה

מה זה Unit Testing ? מה זה שונה מבדיקה ידניות? למה צריך את זה? איך לעשות את זה ? כדאי לעשות TDD? 

**על כל השאלות האלה אני לא אענה באופן מלא.** אם אתם רוצים עוד מידע, סליחה על האכזבה :)

אפשר לבדוק את האפליקציות אנגולר שלנו מ0 על ידי כתיבה והרצה של Pure Javascript functions.

במאמר זה אני אגע ברוב השימושים של בדיקות היחידה (unit test) לאפליקציות אנגולר כמו קומפוננטות, Services, בקשות Http, דירקטיבים, ראוטים, Pipes ועוד!

הבדיקות בAngular השתפרו **המון**! הצוות רשמי של אנגולר עבדו קשה בשביל להוריד Boilerplate ואפשר להגיד שבדיקות באנגולר מעולם לא היו כל כך קלות וטובות. אני משתמש [בJasmine](https://jasmine.github.io/), אבל אפשר גם להשתמש [בMocha](https://mochajs.org/).

### Jasmine

יסמין (אאמץ את השם הישראלי) - לא מדובר על הבת זוג של אלאדין אלא על framework לבדיקות שתומכת בפרקטיקה של BDD - [Behavior-driven development](https://en.wikipedia.org/wiki/Behavior-driven_development). 

לא להבהל, יסמין (ובכללי BDD) מנסה להסביר בדיקות בצורה שניתנת לקריאה בקלות על ידי אנשים ככה שאפילו אנשים לא טכנים יכולים להבין מה נבדק. גם לאנשים טכנים היא מסייעת, כי זה גורם להם להבין ביתר קלות מה קורה בבדיקות. בואו נתחיל בדוגמאות. נניח ואנחנו רוצים לבדוק את הפונקציה הזאת:

```typescript
function angularTesting(): string {
    return 'Testing Rocks!';
}
```

והקוד של יסמין יראה ככה:

```typescript
describe('Angular Testing', () => {  //1
  it('says the truth about', () => {  //2
    expect(angularTesting())  //3
        .toEqual('Testing Rocks!'); //4 
  });
});
```



1. הפונקציה describe מגדירה Test Suite, בעצם אוסף של בדיקות. מקבלת מחרוזת ופונקציה. 
2. הפונקציה it מגדירה בדיקה אחת, המכילה Expectations - אחת או יותר. מקבלת מחרוזת ופונקציה.
3. הביטוי expect (הצפוי) הוא Expectation (ציפייה). ביחד עם ה Matcher הוא מתאר פיסה של התנהגות בתוך האפליקציה. 
4. Matcher הוא ביטוי שעושה השוואה בוליאנית עם הערך ה״צפוי״ מול הערך האמיתי שהועבר לתוך הפונקציה expect. אם הם שליליים הבדיקה נופלת. אפשר לראות את הרשימה המלאה [כאן](https://jasmine.github.io/api/edge/matchers.html). 

### Setup and teardown

בשביל להריץ בדיקה על פיצ׳ר מסויים, אנחנו צריכים לבנות אובייקט בדיקה, לעשות ״נקיון״ אחרי שסיימנו לבדוק ואולי אפילו למחוץ קבצים מהדיסק הקשיח. 

הפעולות האלה נקראות Setup (בנייה) וTeardown (הריסה) וליסמין יש מס׳ פונקציות שיכולות לעשות את החיים שלנו קלים יותר:

1. **beforeAll**

   הפונקציה נקראת פעם אחת לפני שכל הבדיקות בתוך הdescribe רצות. 

2. **afterAll**

   הפונקציה נקראת פעם אחת אחרי שכל הבדיקות בתוך הdescribe רצות. 

3. **beforeEach**

   הפונקציה נקראת לפני כל ריצה של פונקציית it בתוך הdescribe. 

4. **afterEach**

   הפונקציה נקראת אחרי כל ריצה של פונקציית it בתוך הdescribe. 

בשביל להמנע משיכפול קוד בבדיקות שלנו, פרקטיקה טובה היא להשתמש בsetup בשביל קוד שיחזור. לדוגמא, אם נרצה להגדיר משתנים מסויים בתוך הבדיקה. 



### בדיקות והCLI - יחסי אהבה ללא שנאה

כשאנחנו יוצרים פרוייקט עם ה CLI  (אני מקווה מאד שכולם עושים את זה), יווצרו לנו באופן דיפולטיבי קבצי טסטים בעזרת יסמין  וKarama (קארמה). כאשר נייצר קבצים בעזרת הCLI יווצרו קבצים נוספים עם הסיומת spec, שהם בעצם קבצי בדיקות פשוטות. הבדיקות יכילו קצת קוד ראשוני בהתאם לסוג הקובץ שיצרתם: קומפוננטה, פייפ, service וכו׳. 

### הרצת בדיקות בAngular

למזלנו, הCLI של אנגולר מגיע עם פקודה להרצה של בדיקות עם [Karama](http://karma-runner.github.io/0.13/index.html). הCLI מטפל לנו בכל הקונפיגורציות אז בשביל להריץ בדיקה, פשוט הקליקו

```
ng test
```

או, 

```
npm run test
```

וכבר יתווספו לכם watchers על כל שינוי של הקבצים. 



### בדיקות מבוטלות / בפוקוס

אפשר לבטל בדיקות מבלי לשים אותה בהערה, רק עם הוספת האות x לפני הפונקציה describe או לפני הפוקנציה it.

```typescript
xdescribe('Angular Testing', () => {  //1
  xit('says the truth about', () => {  //1
    expect(angularTesting()) 
        .toEqual('Testing Rocks!');
  });
});
```



1. הבדיקות האלה לא ירוצו.

באותה צורה, אפשר לגרום לבדיקות להיות בפוקוס. כלומר, רק הן ירוצו:

```typescript
fdescribe('Angular Testing', () => {  //1
 f it('says the truth about', () => {  //1
    expect(angularTesting())  
        .toEqual('Testing Rocks!'); 
  });
});
```

1, מתוך כל הבדיקות, רק אלו ירוצו. 



### סיכום

יסמין היא framework לבדיקות שתומכת בBDD. אנחנו כותבים בדיקות באוסף של בדיקות (Test Suits) שמחוברות לTest Spec אחד או יותר, שבעצמם מחוברות לTest Expectation אחד או יותר. 

אנחנו מריצים את הבדיקות דרך הדפדפן בעזרת כלי בשם קארמה. קארמה יוצר לנו קבצי HTML, פותח לנו את הדפדפן, מריץ את הבדיקות ומציג את התוצאות. בעזרת Angualr CLI אפשר להריץ את הבדיקות בקלות, על ידי הקלקת ng test. 