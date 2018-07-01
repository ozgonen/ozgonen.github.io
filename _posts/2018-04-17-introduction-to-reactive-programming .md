---
layout: post
title: איך לעבוד עם Observables ולהבין(!!!)
description: "מובלבלים? אנסה לעשות לכם קצת סדר בבלאגן הזה."
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [RxJS]
image: '/images/posts/rxjs.jpg'

---
כשגרסא מספר 2 של אנגולר יצאה (והפכה להיות Angular, כאשר גרסא מס׳ 1 הפכה להיות angular.js) נוסף לחיינו יצור חדש בשם Observables. זה לא פיצ׳ר ספציפי של אנגולר אלא סטנדרט חדש לניהול מידע אסיכנורני, שכנראה מתישהו יתווסף לתקן ([אפשר לראות את ההצעה כאן](https://github.com/tc39/proposal-observable)). אנגולר משתמש בObservables בצורה נרחבת במערכת האירועים ובבקשות הHTTP. כל הדוגמאות הרשמיות והדוקומנטציה מדברות ב Observables, ככה שגם אם בחרתם להשתמש בשיטות אחרות במערכת שלכם, [זו הדרך המומלצת ביותר על ידי הצוות הרשמי של אנגולר.](https://angular.io/guide/observables) 

לעשות שינוי כזה גדול בצורת חשיבה יכולה להיות פעולה מאתגרת, אבל אני אנסה להסביר איך לעבוד עם Observables בצורה קלה.

### Observables are lazy Push collections of multiple values

([מתוך הדוקומנטציה של rxjs.js)](https://github.com/ReactiveX/rxjs/blob/master/doc/observable.md)

אהה… אוקיי. זה יחסית ברור. בוא נחלק את המשפט לשני חלקים:

1. **Observables are lazy Push collections**

   עוזר לי לחשוב על Observables כמו על ניוזלטר. עבור כל אחד שנרשם, ניוזלטר חדש נוצר. וכאשר מופץ דיוור חדש, כל מי שנרשם מקבל את הניוזלטר ורק הם.

2. **Observables יכולים לקבל מס׳ ערכים במהלך הזמן**

   אם נמשיך בדוגמא של ניוזלטר - אם אני אמשיך את המנוי שלי (subscription) לניוזלטר, אני אמשיך לקבל דיוורים מדי פעם ובמשך תקופה. מי שאחראי על הניוזלטר מחליט מתי אני אקבל את הדיורים אבל כל מה שאני צריך לעשות זה פשוט לחכות עד שהדיור יגיע לתיבת המייל שלי.



אם אתם מגיעים מעולם של פרומיסים (Promise), חשוב להבין ההבדל העיקרי הוא שפרומיס תמיד מחזיר רק ערך אחד. הבדל נוסף שקיים, הוא שObservables הם ברי ביטול. אם אני לא רוצה להמשיך לקבל דיוור, אני יכול לבטל את המינוי שלי. פרומיסים שונים בכך שאי אפשר לבטל אותם - אם קיבלת פורמיס, התהליך של התמודדות (Resolve) עם הפרומיס יקרה, ואין לך אפשרות לבטל את הביצוע שלו. 



### לדחוף (Push) לעומת לקבל (Pull)

באופן דיי טריויאלי, ניתן להגדיר את הצורה שבה צרכני מידע ויצרני מידע משוחחים אחד עם השני בעזרת **דחיפה** ובעזרת **קבלה**.

**קבלה**

כאשר צרכן מידע מקבל מידע, הוא מחליט מתי לקבל אותו ויצרן המידע לא באמת מודע לזמן המדוייק שבו ההמידע מתקבל.

לדוגמא, כל פונקציית ג׳אווסקריפט היא יצרנית מידע והקוד שקורא לפונקציה צורך אותה על ידי **קבלת** ערך אחד בודד.

**דחיפה**

כשדוחפים מידע, זה דיי טריוואלי שזה עובד בצורה הפוכה. יצרן המידע מחליט מתי צרכן המידע יקבל את המידע. בדיוק כמו בפרומיסים, הפרומיס מעביר מידע לתוך הפונקציית קולבאק, אבל רק כאשר הוא מחליט שהמידע מוכן להעברה. 

Observables זו שיטת חדשה לדחוף מידע. Observable הוא יצרן של מס׳ ערכים ודוחף אותם לצרכני מידע ש״נרשמו״ (subscribe) לשירות קבלת המידע הזה. 



### Observables באנגולר (סוף סוף!)

כנראה שהמפגש הראשון שלך עם Observables, יהיה עם בקשות HTTP.

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { User } from './users/model/user'
@Injectable()
export class AppService {

  constructor(private http: HttpClient) { }

  getUsers(): Observable<User[]> {
    const url = `https://jsonplaceholder.typicode.com/users`;
    return this.http.get(url).pipe(map((response: User[]) => response));
  };

}
```



יצרתי סרביס פשוט מאד, שמחזיר Observable(ביחיד) כשאני אקרא למטודה getUsers. המטודה תחזיר Observable מסוג מערך של משתנים.

 סביר להניח שארצה להציג את שמות המשתמשים באיזשהי צורה - אז נצטרך להרשם אל המטודה (subscribe). נוכל לעשות את זה בשתי דרכים:

1. **Async Pipe**

   אפשר להרשם לObservables דרך הטמפלט של אנגולר, בעזרת Async Pipe. הייתרון המשמעותי זה שאנגולר מנהל עבורנו את כל ההרשמה במהלך כל אורך החיים (Life Cycle) של הקומפוננטה. אנגולר ירשם ויבטל את ההרשמה עבורנו שזה ממש כיף.

   

   ```ts
   import { Component, OnInit } from '@angular/core';
   import { AppService } from '../app.service';
   import { Observable } from 'rxjs';
   import { User } from './model/user'
   
   @Component({
     selector: 'app-users',
     template: `
   <ul class="userList" *ngIf="(users$ | async)?.length > 0">
         <li class="user" *ngFor="let user of users$ | async">
           {{ user.name }} - {{ user.email }}
       </li>
   </ul>
     `
   })
   export class UsersComponent implements OnInit {
   
     constructor(private appService: AppService) { }
   
     public users$: Observable<User[]>
     ngOnInit() {
       this.users$ = this.appService.getUsers();
     }
   }
   ```

   שימו לב לסימן הדולר $ ! זה מסמן לנו שהמשתנה הוא Observable וזה נחשב Best Practive. בדרך הזאת, קבל לזהות שהמשתנה הוא Observable או לא.

2. **צורה ידנית**

   אנחנו נרשם בעצמנו לObservable בעזרת המטודה subscribe. נרצה להשתמש בשיטה זו אם נרצה לערוך את המידע לפני שנרצה להציג אותו. החסרון הוא שנצטרך לנהל את ההרשמות בעצמנו. 

```ts
import { Component, OnInit } from '@angular/core';
import { User } from '../users/model/user'
import { AppService } from '../app.service';

@Component({
  selector: 'app-users-2',
  template: `
<ul class="userList" *ngIf="users?.length > 0">
      <li class="user" *ngFor="let user of users">
        {{ user.name }} - {{ user.email }}
    </li>
</ul>
  `,
})
export class Users2Component implements OnInit {

  public users: User[];

  constructor(private appService: AppService) { }

  ngOnInit() {
    this.appService.getUsers()
      .subscribe((data: User[]) => this.users = data)
  }
}
```



כמו שאפשר לראות, הלוגיקיה של הטמפלט היא יחסית דומה. השוני העיקרי הוא בלוגיקה של הקומפוננטה, שיכולה להסתבך אם נבחר באפשרות מס׳ 2. חדי העין ישימו לב שלא טיפלתי בביטול ההרשמה. למען האמת, אין צורך כיוון שהשתמשתי ב Http Client  של אנגולר, ו[הוא עוזר לי בנושא הזה.](https://stackoverflow.com/questions/35042929/ist-it-necessary-to-unsubscribe-from-observables-created-by-http-methods) 



### יצירת Observable

אחרי שלמדנו להתמודד עם Observables נפוצים שניתנים על ידי אנגולר, נלמד איך ליצור Observables. הגרסא הפשוטה נראית ככה: 

```ts
import { Component, OnInit } from '@angular/core';
import { Observable } from "rxjs"
import { take } from 'rxjs/operators';

@Component({
  selector: 'app-create-observable',
  template: ``,
})
export class CreateObservableComponent implements OnInit {

  constructor() { }

  ngOnInit() {
    const observable = new Observable((observer) => {
      observer.next("My First Observable")
      observer.complete()
    });
    observable
    .pipe(take(1))
    .subscribe(data => console.log(data))
  }
}
```

אפשר לראות  שיצרנו Observable פשוט, נרשמנו אליו ואז על ידי שימוש במטודה next, דחפנו סטרינג והוא הגיע לsubscribe שיצרנו, והוא הודפס. בגלל שהשתמשתי באופרטור take, אנחנו מפסיקים להאזין לObservable ובעצם ככה ביטלנו את ההרשמה. 

??? **לאט לאט.** 



**יצירת Observable** היא פשוטה. פשוט קוראים למטודה new Observable ומעבירים ארגומנט שהוא יהיה הobserver. 

**הרשמה ל Observable** אתם זוכרים שObservables הם Lazy? אם אנחנו לא נירשם, שום דבר לא יקרה. בכל פעם שנרשם לObservable - בכל קריאה של subscribe תבוצע הפעלה של ה observable. זה ממש דומה לזה שאם אני נרשם לרשימת דיוור, אז מיד אני אפעיל את מנגון הדיוור, ואני אקבל מייל.  

**ביצוע של Observable** הקוד בתוך ה Observable עצמו, מחולק ל3 פונקציות:

1. next - שולח כל ערך למי שנרשם אליו.
2. Error - מעביר שגיאת ג׳אווה סקריפט או exception.
3. Complete  - לא שולח אף ערך.

קריאות של next הן הנפוצות ביותר, כיוון שהם מעבירות את המידע לאלו הרשומים. בתוך Observable יכולות להיות אין סוף קריאות לnext אבל רק error או complete אחד, כיוון שהם יעצרו את הפעולה של הObservable ואף מידע לא ישלח. 

חדי העין יעירו ובצדק, שאין צורך היה לבטל את ההרשמה בדוגמא, על ידי שימוש באופרטור take כיוון שהObservable קורא לפונקציה complete, מיד אחרי הקריאה לפונקציה next. קריאת Http Client של אנגולר עובדת באותה צורה. 



### **סיכום**

את כל הדוגמאות קוד ניתן להוריד [מכאן](https://stackblitz.com/edit/working-with-observables).

הפוסט הזה היה אמור לתת לכם הבנה קצת יותר טובה על איך Observables עובדים. יש עוד **המון** ללמוד ולהבין, והשלב הבא הוא להתמקצע באופרטורים ובrxjs. בפוסט הבא, אפרסם דוגמא אמיתית לאיך אפשר להשתמש בכל הקסם הזה. 

מוזמנים להשאיר תגובה / לשאול עוד שאלות.





