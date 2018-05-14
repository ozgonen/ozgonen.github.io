---
layout: post
title: Introduction to Reactive Programming
description: "איזה פחד. כשדיברתי עם אנשים סביבו על הפרדיגמה הזאת, אנשים שיקשקו מרוב פחד."
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [RxJS]
image: '/images/posts/rxjs.jpg'

---
תיכנות ריאקטבי.

איזה פחד. כשדיברתי עם אנשים סביבו על הפרדיגמה הזאת, אנשים שיקשקו מרוב פחד. ״זו פרידגמה מתקדמת! זה כל כך מסובך! זה מאד מבלבל! עוד לא הצלחנו להתאושש מלכתוב תכנות פונקציונלי אחרי שהתרגלנו לתכנות מונחה עצמים, ועכשיו אתם מדברים איתנו על Reactive?״

את האמת? גם אני חשבתי ככה. אבל הבנתי בסופו של דבר שהדבר הכי קשה בתכנות ריאקטיבי זה לחשוב בצורה ריאקטיבית. בפוסט הזה אני אנסה ללמד לחשוב בצורה ריאקטיבית בעזרת RxJS. אם אתם חדשים לגמרי בתחום של Reactive, ממליץ לכם לקרוא את [הפוסט הקודם שלי בנושא](http://www.ozgonen.co.il/asynchronous-programming).



### מה זה תכנות ריאקטיבי ?

בתאכלס זה לתכנת עם Stream של אירועים. סדרה של אירועים שמתרחשים במהלך זמן. כמו בעצם מערך של איברים, שכל איבר מתווסף בצורה אסינכורנית. נוכל להאזין למערך הזה, וכל פעם שיתווסף אירוע חדש למערך נוכל לעשות איתו משהו. **זה הרעיון המרכזי.**

בוא נעשה השוואה פשוטה בין מערך לבין Observable. נניח ויש לנו מערך ואנחנו רוצים לעבור עליו ולהדפיס את כל האיברים שלו. כל המערך והאיברים שלו נמצאים בזיכרון, ואנחנו בצורה פשוטה עוברים עליו ומדפיסים את האיברים.

```ts
const davidBeckhamNumbers: number[] = [28, 15, 24, 7, 10, 23, 32];

function printNumbers(numbers: number[]): void {
  numbers.map(number => console.log(number));
}
```

לעומת זאת, Stream של אירועים לא נמצא בזיכרון. האירועים קורים במהלך הזמן ואנחנו לא באמת יודעים מתי הם יקרו. 

(לא לבהל מהקוד!)

```typescript
import * as Rx from 'rxjs';

const davidBeckhamNumbers = Rx.Observable
    .interval(400)
    .take(7)
    .map(i => [28, 15, 24, 7, 10, 23, 32][i]);

davidBeckhamNumbers.subscribe(number => console.log(number));
```

יש לנו את אותם איברים כמו שהיו במערך, אבל האיברים אלה מגיעים במהלך זמן, כל 400 מילי שניות. הוספתי Event Listener בשם subscribe, וכל פעם שאירוע קורה - אני מדפיס אותו. 

בנוסף, זה אפשר להשתמש באותם פונקציות של מערכים גם על Stream של אירועים. 

נניח ואנחנו רוצים לסכום את כל המספרים. אבל במערך שלנו מגיעים גם מחרוזות, וגם מספרים בצורת מחרוזות. 

```typescript
const davidBeckhamNumbers: any[] = [28, 15, '24', 7, 'GGMU', 10, 23, 32, 'oz'];

const numbersSum =
          davidBeckhamNumbers
              .map(x => parseInt(x))
              .filter(x => !isNaN(x))
              .reduce((prev, current) => prev + current);

console.log(numbersSum);
```

אנחנו עוברים איבר איבר במערך: הופכים אותו למספר, מפלטרים מספרים לא חוקיים וסוכמים (sum). 

הדבר המדהים הוא, שאותו קוד בדיוק יעבוד גם על Stream של אירועים, ולא משנה אם הם מגיעים בזמן שונה או שכולם זמינים בזיכרון.

```typescript
import * as Rx from 'rxjs';

const davidBeckhamNumbers = Rx.Observable
    .interval(400)
    .take(7)
    .map(i =>  [28, 15, '24', 7, 'GGMU', 10, 23, 32, 'oz'][i]);

const numbersSum =
          davidBeckhamNumbers
              .map(x => parseInt(x))
              .filter(x => !isNaN(x))
              .reduce((prev, current) => prev + current);

numbersSum.subscribe(number => console.log(number));
```

### למה לבחור ב RxJS ?

עבורי, הסיבה הברורה ביותר היא שRxJS מאפשר להגדיר את ההתנהגות הדינמית של ערך באופן מלא, בזמן ההצהרה. 

יצא קצת משפט סבוך, אבל אני אנסה להסביר בעזרת קוד. 



```typescript
let a = 5;
let b = 6 * a;

console.log(b) // 30

a = 4;
console.log(b) // 30
```

בהתחלה הגדרתי את משתנה a עם המספר 5, ואת המשתנה b כתלות במשתנה a. ובאמת שהדפסתי את b, קיבלתי את התשובה הנכונה - 30. אחר כך, החלפתי את a, והדפסתי את b. אנחנו יודעים שהתשובה היא 30, אבל היינו רוצים את הדינמיות הזאת. בהצהרה של משתנה b, הגדרנו אותו כפי שש מa, למרות שהיינו מצפים שהערך של b ישתנה עם הזמן, כי הוא לא קבוע (כתלות במשתנה אחר) - הערך שלו נשאר אותו ערך כמו בזמן ההצהרה. 

ניתן להשיג את זה עם Stream של אירועים:

```typescript
let streamA = Rx.Observable.of(5, 6);
let streamB = streamA.map(a => 6 * a);

streamB.subscribe( b => console.log(b));
```

בגלל שהגדרנו את ההתנהגות הדימנית של הערך a, הערך של b מושפע מכל אירוע שיגיע. לא רק צריך להגדיר את ההתנהגות של b פעם אחת אלא גם את a. 



המשך בפוסט מס׳ 2 :)

