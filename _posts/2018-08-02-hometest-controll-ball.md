---
layout: post
title: פתרון מבחן בית מס׳ 1 - להעביר מידע בין קומפוננטות
description: "בואו מעביר מקומפוננטה אחות וישר נציג את המידע"
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [Home Tests]
image: '/images/posts/2018-02-02-hometest-controll-ball.jpg'

---



חשבתי שיהיה מעניין, לנסות להעלות לכאן פתרונות של מבחני בית שמטרתם היא לסנן מרואיניים בתהליך גיוס. 

אספתי במהלך השנים מבחנים שאני עברתי, מבחנים שחברים עברו ומבחנים שמצאתי בחיפושים באינטרנט. אני חושב ששיתוף פתרונות יוכל להיות מועיל בהרבה תחומים שכן מצד אחד, מציג בעיות ״עולם אמיתי״ ומצד שני יכול לסייע לכם לראות איך אני נגשתי לפתרונות. כמובן שאם אתם חושבים אחרת ממני, אשמח מאד (!!) לפידבק. 



בחרתי להתחיל עם מבחן בית מאד קל, במקור הוא נועד לכתיבה בג׳אווה סקריפט ונילה - קצת שידרגתי אותו לעולם שלנו. 

אז בואו נתחיל. 

#### מבחן בית מס׳ 1 - שליטה בכדור

![juggle](/images/gifs/juggle.gif)

**רמת קושי:** מתכנת מתחיל

**טכנולוגיות:** אנגולר

**תיאור**: צרו אפליקציה שמחולקת לקומפוננטות. מבחינה ויזואלית, האפליקציה צריכה להידמות ל״טלווזיה״ במובן שבקומפוננטה הראשית (המסך)  יש עיגול (כדור) ובשניה יש כפתורים (כמו טלוויזיה ישנה). הכפתורים הם ״למעלה״, ״למטה״, ״ימינה״, ״שמאלה״, ״איפוס״. המצב ההתחלתי של האפליקציה הוא שהעיגול נמצא במרכז המסך. 

1. כאשר המשתמש ילחץ על אחד מהכפתורים, העיגול יזוז לאותו כיוון. כאשר המשתמש ילחץ על איפוס, הכדור יחזור למיקום ההתחלתי.
2. הכדור לא יכול לצאת מגבולות המסך.



### תכנון

תמיד שאני ניגש למטלה כזו, או שאני עובד על פיצ׳ר חדש, אני מתחיל לתכנן את הקומפוננטות שלי ואת הממשק שלהם. 

במקרה הזה, זה יחסית ברור לי: מדובר על שתי קומפוננטות אחיות שהן המסך (screen) והשלט (control).

המסך יחזיק בתוכו את הלוגיקה של הכדור, והשלט יחזיק את הלוגיקיה של הכפתורים. הקומפוננטות ידברו בינהם דרך הקומפוננטת אב (app) שתקבל event כל פעם שנלחץ כפתור מקומפוננטת השלט, ותשנה אובייקט שיועבר לקומפוננטת המסך, שיזיז את הכדור.



### ביצוע

בואו נצלול לקוד.

הקומפוננטת אבא, תחזיק את הילדים כמו שאמרנו:

**app.component.html**

```html
<app-screen
  [ngStyle]="{'width' : screenWidth, 'height': screenHeight}"
  [moveBall]="newBallDirection"
></app-screen>
<app-control
  (moveBall)="moveBall($event)"
></app-control>
```

שימו לב, שהגדרתי את גובה המסך ורוחב המסך כמשתנים. הסיבה היא מאד פשוטה, אנחנו נרצה לשלוט בצורה דינמית בגבולות של המסך ובהתאם שנוכל להגביל את התזוזה של הכדור.

עכשיו ניגש להגדיר את ההתנהגות, שתהיה כשנרצה להזיז את הכדור

**app.component.ts**

```typescript
import {Component} from '@angular/core';
import {Direction} from './model/direction';
import {BallDirections} from './model/ball-directions';
import {maxHeight, maxWidth} from './consts/consts';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styles: [
      `:host {
      display: flex;
      margin: 10vh 0;
      width: 80%;
    }`
  ]
})
export class AppComponent {
  newBallDirection: BallDirections;
  screenWidth = `${maxWidth}px`;
  screenHeight = `${maxHeight}px`;

  moveBall(direction: Direction) {
    this.newBallDirection = {direction};
  }
}
```

כאשר המשתמש לוחץ על כפתור כלשהו, הפונקציה  moveBall תקרא, עם הכיוון החדש. ידרס האובייקט newBallDirection שהגדרתי על הקומפוננטה עם אובייקט חדש, והוא יועבר לקומפוננטה screen. בחרתי במבנה נתונים של אובייקט כי בכל דריסה כזו יווצר פויינטר חדש והקומפוננטה תדע שהתקבל אובייקט חדש.  

**screen.component.ts**



```typescript
import {Component, Input} from '@angular/core';
import {BallDirections} from '../model/ball-directions';
import {horizontalSteps, moveAmount, verticalSteps} from '../consts/consts';

@Component({
  selector: 'app-screen',
  template: `
    <div
      [ngStyle]="ballDirections"
      class="ball"
    ></div>
  `,
  styleUrls: ['./screen.component.scss']
})
export class ScreenComponent {
  ballDirections = {
    marginTop: '0',
    marginLeft: '0'
  };

  @Input() set moveBall(value: BallDirections) {
    if (value) {
      this.ballDirections = this.moveTheBall(value);
    }
  }

  /**
   * Changes the ball directions object
   * by the object
   * @param value
   */
  private moveTheBall(value: BallDirections) {
    let {marginTop, marginLeft} = this.ballDirections;
    const top: number = parseInt(marginTop, 10);
    const left: number = parseInt(marginLeft, 10);
    switch (value.direction) {
      case 'up':
        if (top > -Math.abs(moveAmount * verticalSteps)) {
          marginTop = `${top - moveAmount}px`;
        }
        break;
      case 'down':
        if (top < moveAmount * verticalSteps) {
          marginTop = `${top + moveAmount}px`;
        }
        break;
      case 'left':
        if (left > -Math.abs(moveAmount * horizontalSteps)) {
          marginLeft = `${left - moveAmount}px`;
        }
        break;
      case 'right':
        if (left < moveAmount * horizontalSteps) {
          marginLeft = `${left + moveAmount}px`;
        }
        break;
      case 'reset':
        marginLeft = '0px';
        marginTop = '0px';
        break;

    }

    return {marginTop, marginLeft};
  }


}
```

על כל שינוי באובייקט הכיוונים, ישנה קריאה לפונקציה שבונה אובייקט של css styles וככה בעצם מזיזה את הכדור. הפונקציה מתחשבת כמובן בstate הנוכחי של הכדור ומונעת ממנו לזוז אם הוא הגיע לקצה המסך. אם הגיעה הפקודה לאיפוס, גם הstyle מתאפס. 

**control.component.ts**

כל שנותר להגדיר את ה״שלט״. סה״כ מעביר את המחרוזת שמתארת את הלחיצה של המשתמש. כל שאר ההצהרות הן עבור האייקונים הנכונים.



```typescript
import {Component, EventEmitter, Output} from '@angular/core';
import {faArrowAltCircleUp, faArrowCircleDown, faArrowCircleLeft, faArrowCircleRight, faUndoAlt} from '@fortawesome/free-solid-svg-icons';
import {Direction} from '../model/direction';

@Component({
  selector: 'app-control',
  templateUrl: './control.component.html',
  styleUrls: ['./control.component.css']
})
export class ControlComponent {
  up = faArrowAltCircleUp;
  down = faArrowCircleDown;
  left = faArrowCircleLeft;
  right = faArrowCircleRight;
  undo = faUndoAlt;

  @Output() moveBall = new EventEmitter<string>();

  onButtonClick(direction: Direction) {
    this.moveBall.emit(direction);
  }

}
```

**control.component.html**

```html
<div class="buttons">
  <div class="up">
    <fa-icon [icon]="up" (click)="onButtonClick('up')"></fa-icon>
  </div>
  <div class="directions">
    <fa-icon [icon]="left" (click)="onButtonClick('left')"></fa-icon>
    <fa-icon [icon]="right" (click)="onButtonClick('right')"></fa-icon>
  </div>
  <div class="down">
    <fa-icon [icon]="down" (click)="onButtonClick('down')"></fa-icon>
  </div>


  <div class="undo">
    <fa-icon [icon]="undo" (click)="onButtonClick('reset')"></fa-icon>
  </div>
</div>
```

### תוצאות



![movingBall](/images/gifs/movingBall.gif)

אפשר לראות את כל הקוד [כאן](https://github.com/ozgonen/MovingBall) או [כאן](https://stackblitz.com/github/ozgonen/MovingBall) 

ואפשר לראות את התוצאה [ישר כאן](https://rlmiqygw.github.stackblitz.io)

