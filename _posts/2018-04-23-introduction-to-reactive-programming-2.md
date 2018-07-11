---
layout: post
title: תכנות ריאקטיבי - הלכה למעשה
description: "מה שממש קשה להבין מכל החפירה שעשיתי זה איך לעזאזל עובדים עם זה."
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [RxJS]
image: '/images/posts/rxjs.jpg'

---

זה פוסט המשך של הפוסט הקודם :)

אם הגעתם לכאן בטעות, תקראו את הפוסט הקודם.

### תכנות ריאקטיבי - הלכה למעשה

מה שממש קשה להבין מכל החפירה שעשיתי במשך שתי פוסטים, אז איך לעזאזל עובדים עם זה. אז אני אנסה להסביר איך באמת עובדים עם תכנות ריאקטיבי בעולם האמיתי. אני הולך להציג איך אני משתמש RxJS בשביל תקשורת בסיסית בין שרת - לצד לקוח בשילוב עם TypeScript. 

בדוגמא שנציג, נשתמש ב https://www.pokeapi.co ונממש פונקצונליות של מידע עם Pagination:

1. בזמן טעינה נטען 5 פוקימונים.
2. בזמן שנלחץ על כפתור Random יגיעו פוקימונים רנדומלים.
3. בזמן שנלחץ על חץ קדימה, יגיעו ה5 פוקימונים הבאים. בזמן שנלחץ על חץ אחורה, יגיע ה5 פוקימונים קודמים. (אם אפשר) 



דברים חשובים:

1. אני משתמש בhttps://stackblitz.com כי הוא סופר נוח לשיתוף קוד בין כולם. אם לא הכרתם - הגיע הזמן!
2. אני אשתמש ב [httpClient](https://angular.io/guide/http) של אנגולר ולא בספריות אחרות - לא כי זו ספריה מעולה, אבל בעיקר כי היא דה פקטו של אנגולר והדיפולט שלה הוא בהחזרה של זרם מידע.
3.  הקוד המלא נמצא [כאן](https://stackblitz.com/edit/ozgonen-pokemon-example) אם בא לכם להציץ.



### הכנות לפני:

יצרתי סרביס לאפליקציה, שהוא מחזיק את הקריאת API דרך HttpClient. בגלל שהצצתי קצת בדוקומנטציה של pokeapi, ראיתי שיהיה לי נוח להגדיר את הURL בהתאם לתוצאות של התשובה. כל תשובה מחזיקה משתנים בשם next וprevious ויהיה לי מאד קל להעביר אותם לסרביס. הגדרתי גם ערך דיפלוטיבי למשתנה, בשביל הקריאה הראשונה. 

app.service.ts

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class AppService {

  constructor(private http: HttpClient) { }

  getPokemon(url) {
    url = url ? url : `https://pokeapi.co/api/v2/pokemon/?limit=5`;
     return this.http.get(url);
  };

}
```

### בקשות ותשובות

 בואו נתחיל שהכל (כמעט) יכול להיות זרם מידע. יצרתי קומפוננטה חדשה בשם app-pokemon ואני ארצה לעשות קריאת API ברגע שהקומפוננטה נטענת. שימו לב שאני כותב את הקריאה בngOnInit ולא בconstractor. 

אין פה שום דבר מיוחד. (1) מבצעים בקשה (2) מקבלים תשובה (3) מרדנדרים את התשובה.

בגלל שאנחנו משתמשים בHttpClient הבקשות שלנו מיוצגות כבר בתור זרם מידע. 

```typescript
import { Component, OnInit } from '@angular/core';
import { AppService } from '../app.service';
import { ApiResponse } from './model/api-response';
@Component({
  selector: 'app-pokemon',
  templateUrl: './pokemon.component.html',
  styleUrls: ['./pokemon.component.css']
})
export class PokemonComponent implements OnInit {

  pokemonData: ApiResponse;
  constructor(private appService: AppService) { }
  ngOnInit() {
    this.fetchData();
  }

  private fetchData(metaData = null) {
    this.appService
      .getPokemon(metaData)
      .subscribe(data => this.handleResponse(<ApiResponse>data));
  }

  handleResponse(data: ApiResponse) {
    this.pokemonData = data;
  }

}

```



יצרתי subscribe לObservable (המינוח הרשמי לזרם מידע) בעצם נוצר זרם מידע מתוך השרת כתגובה ויש עידכון כל פעם שיש משהו חדש - במקרה אנחנו מדפיסים את התשובה. אנחנו יוצרים בקשה אחת - ככה שאם המודל הוא זרם של מידע, יתקבל רק ערך אחד. שימו לב שהשתמשתי באופרטור take. מה שהוא עושה זה פשוט, אחרי שהתקבלה התשובה הוא מפסיק להאזין לObservable - אנחנו לא צופים שיגיע זרם חדש של מידע.  השלב הבא הוא להתמודד עם התוצאה ולרדנדר אותה:

```html
<button #refresh>Refresh</button>
<div *ngIf="pokemonData">
  <ul>
    <li *ngFor="let result of pokemonData.results">
      {{result.name}}
    </li>
  </ul>
</div>
```

### כפתור Random

כל פעם שהכפתור נלחץ, אנחנו אמורים ליצור כתובת url חדשה בשביל שנוכל לקבל reponse חדש. ניצור offset רנדומלי ואיתו נקבל רשימה חדשה רנדומלית. אנחנו צריכים 2 דברים: Observable של לחיצות על הכפתור וליצור בקשה חדשה שתשנה את המידע שאותו אנחנו מרנדרנים בהתאמה ל Observable של לחיצות הכפתור. 

קודם כל אוסיף כפתור בטמפלט עם משתנה לוקאלי (#) בשביל ViewChild:

```html
<button #refresh>Refresh</button>

<div *ngIf="pokemonData">
  <ul>
    <li *ngFor="let result of pokemonData.results">
      {{result.name}}
    </li>
  </ul>
</div>
```



אצור מטודה חדשה שתטפל בכפתור Refresh:

```typescript
  private initRefreshButton() {
    Observable
      .fromEvent(this.refreshButton.nativeElement, 'click')
      .map(data => this.createNewUrl())
      .subscribe(data => this.fetchData(data));
  }
  private createNewUrl() {
    const randomOffset = Math.floor(Math.random() * 500);
    return `https://pokeapi.co/api/v2/pokemon/offset=${randomOffset}`;
  }
```

יצרנו Observable מתוך הלחיצה של הכפתור, עם כל לחיצה אני רוצה ליצור url חדש שאותו אני אשלח לשרת. האופרטור Map הוא אותו אופרטור שאנחנו מכירים ממערכים (אפשר לקרוא בהרחבה [בפוסט הזה](http://www.ozgonen.co.il/asynchronous-programming/)) כאשר כל אירוע שקורה, אנחנו ניצור עבורו כתובת url חדשה עם  offset רנדומלי. נירשם על הObservable הזה וכל תשובה נשלח לשרת בשביל לקבל מידע על פוקימונים חדשים. 

**הערה חשובה: כל Observable חייב להפסיק להזין ברגע שהקומפוננטה נעלמת - אחרת הוא ימשיך להאזין ללא הפסקה. נתמודד עם זה בהמשך.** 



### כפתור Next/Prev

אחרי שעשינו את כל ההכנה הזאת, העבודה יחסית פשוטה. נוסיף לטמפלט את שתי הכפתורים עם משתנה לוקאלי (גם נדאג שלא יפעלו במידה הצורך):

```html
<button #refresh>Refresh</button>

<div *ngIf="pokemonData">
  <ul>
    <li *ngFor="let result of pokemonData.results">
      {{result.name}}
    </li>
  </ul>
</div>
<button [disabled]="pokemonData && !pokemonData.previous" #previous>Previous</button>
<button [disabled]="pokemonData && !pokemonData.next" #next>Next</button>
```



ניצור מטודה שתשלוט בלוגיקה של הניווטים. גם היא תקרא בngOnInit. כמו במקרה של כפתור refresh, גם פה אצור Observable לכל אחד מהכפתורים, ואצור url בהתאם לפעולה. אם נלחץ על הכפתור של התוצאות הבאות, נרצה שהObservable יחזיק את הכתובת של התוצאות הבאות ואותו נשלח לAPI.

מיד אחרי יצירת ה Observables, יצרתי Observable נוסף שמאחד בין הObservables שיצרתי ועליו נרשמתי. בצורה זו, רשמתי את לוגיקת הטיפול במקום אחד במקום להאזין לכל אחד מהם בנפרד. 

```typescript
ngOnInit() {
  this.fetchData();
  this.initRefreshButton();
  this.handleNavigationButtons();
}

private handleNavigationButtons() {
  const nextEvents$ = Observable
    .fromEvent(this.nextButton.nativeElement, 'click')
    .map(data => `${this.pokemonData.next}`);

  const previousEvents$ = Observable
    .fromEvent(this.prevButton.nativeElement, 'click')
    .map(data => `${this.pokemonData.previous}`);


  Observable
    .merge(nextEvents$, previousEvents$)
    .subscribe(data => this.fetchData(data));
}
```



### ניקוי

כמו שאמרתי מקודם, חשוב שכל Observable לא ימשיך להאזין אחרי שהקומפוננטה הושמדה. היינו מצפים שזה יקרה אוטומטית, אבל אנחנו צריכים להגדיר את זה באופן ידני. אפשר להשתמש [בדקורטור הזה](https://github.com/NetanelBasal/ngx-take-until-destroy) של [Netanel Basal](https://medium.com/@NetanelBasal) אבל אני מעדיף לעבודה בצורה ידנית (קצת מזכיר  שחרור זיכרון מימי C).

לכל אחד מהObservables נוסיף אופרטור בשם takeUntil, שאומר לו בעצם מתי להפסיק להאזין. נגדיר Subject Observable ונשתמש ב lifehook של Angular, במטודה ngOnDestroy בשביל לקדם אותו. זה קצת חומר מתקדם, שאני אדבר עליו בהמשך אז אם אתם לא מבינים עד הסוף - אין מה לדאוג :) 

והנה כל הקוד של הקומפוננטה:



```typescript
import {Component, ElementRef, OnDestroy, OnInit, ViewChild} from '@angular/core';
import {AppService} from '../app.service';
import {ApiResponse} from './model/api-response';
import {Observable} from 'rxjs/Rx';
import 'rxjs/add/observable/fromEvent';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/take';
import 'rxjs/add/operator/merge';
import {Subject} from 'rxjs/Subject';

@Component({
  selector: 'app-pokemon',
  templateUrl: './pokemon.component.html',
  styleUrls: ['./pokemon.component.css']
})
export class PokemonComponent implements OnInit, OnDestroy {

  @ViewChild('refresh') refreshButton: ElementRef;
  @ViewChild('next') nextButton: ElementRef;
  @ViewChild('previous') prevButton: ElementRef;
  pokemonData: ApiResponse;
  ngUnSubscribe: Subject<void> = new Subject<void>();

  constructor(private appService: AppService) {
  }

  ngOnInit() {
    this.fetchData();
    this.initRefreshButton();
    this.handleNavigationButtons();
  }

  ngOnDestroy() {
    this.ngUnSubscribe.next();
    this.ngUnSubscribe.complete();
  }

  private handleNavigationButtons() {
    const nextEvents$ = Observable
      .fromEvent(this.nextButton.nativeElement, 'click')
      .map(data => `${this.pokemonData.next}`);

    const previousEvents$ = Observable
      .fromEvent(this.prevButton.nativeElement, 'click')
      .map(data => `${this.pokemonData.previous}`);


    Observable
      .merge(nextEvents$, previousEvents$)
      .takeUntil(this.ngUnSubscribe)
      .subscribe(data => this.fetchData(data));
  }

  private fetchData(metaData = null) {
    this.appService
      .getPokemon(metaData)
      .subscribe(data => this.handleResponse(<ApiResponse>data));
  }

  private initRefreshButton() {
    Observable
      .fromEvent(this.refreshButton.nativeElement, 'click')
      .map(data => this.createNewUrl())
      .takeUntil(this.ngUnSubscribe)
      .subscribe(data => this.fetchData(data));
  }

  private handleResponse(data: ApiResponse) {
    this.pokemonData = data;
  }

  private createNewUrl() {
    const randomOffset = Math.floor(Math.random() * 500);
    return `https://pokeapi.co/api/v2/pokemon/?limit=5&offset=${randomOffset}`;
  }
}
```



### לסיכום

אז יצרנו אפליקציה יחסית פשוטה, אבל בסגנון ״עולם אמיתי״. כל הקוד אפשר למצוא [כאן](https://stackblitz.com/edit/ozgonen-pokemon-example). 

אשמח לשאלות / תיקונים :)