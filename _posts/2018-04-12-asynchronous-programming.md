---
layout: post
title: Asynchronous Programming
description: " תכנות אסינכורני יכול להיות מרתיע. איך אפשר לכתוב אפליקציה שתקבל אינפוט ממשתמש, תריץ אנימציות ותשלח בקשות לשרת באותו זמן? איך אפשר להתמודד עם שגיאות אסינכורניות?."
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [RxJS]
image: '/images/posts/asyncProg.jpg'

---
 תכנות אסינכורני יכול להיות מרתיע. איך אפשר לכתוב אפליקציה שתקבל אינפוט ממשתמש, תריץ אנימציות ותשלח בקשות לשרת באותו זמן? איך אפשר להתמודד עם שגיאות אסינכורניות?

לטעמי, אחד החלקים הכי חשובים בלהיות מפתח ווב אפקטיבי זה לבנות ולנהל אפליקציה אסינכורנית.
בשונה מרוב שפות התיכנות, ג׳אווה סקריפט היא single-threaded.
 כתוצאה מכך, אפליקציה ג׳אווה סקריפט חייבות להשתמש בAPI אסינכורני בשביל להשאר רספונסיב למשתמש בעת ביצוע פעולות שנמשכות זמן רב, כמו בקשות לשרת ואנימציות.

 אז הנה קצת חדשות טובות: תיכנות אסינכורני הוא יותר פשוט ממה שזה נראה. המפתח הוא לחשוב בצורה שונה לגבי אירועים - אפשר לבנות את רוב התוכנות האסינכורניות באמצעות פונקציות פשוטות.
  בסדרת פוסטים אנסה להסביר מדוע רוב מפתחי הג׳אווה סקריפט ניגשים לתיכנות אסינכורני בצורה שגוייה, ואיך ניתן להמנע מהטעויות האלה. עד סוף הסדרה, תוכלו להבין את הכלים, קונספטים, וסיפוריות הדרושות להפוך להיות נינג׳ת תיכנות אסינכורני!

 בפוסט זה, אלמד את הסוד הראשון - **תיכנות בלי לולואת**. הלולאות של ג׳אווה סקריפט יכולות לרוץ רק בצורה סינכורנית. כתוצאה מכך, בשביל להיות אלופי תיכנות אסיכורוני, אנחנו צריכים להיות אלופי תיכנות בלי לולאות. 

### forEach / Map
נניח ויש לנו מערך של שחקני כדורגל מפורסמים שקיבלנו מהשרת, ואנחנו רוצים להוציא ממנו רק את השמות של השחקנים.

```typescript
const players: Player[] = [
  {
    name       : 'Cristiano Ronaldo',
    position   : 'Forward',
    shirtNumber: 7
  },
  {
    name       : 'Lionel Messi',
    position   : 'Forward',
    shirtNumber: 10
  },
  {
    name       : 'Eran Levy',
    position   : 'Striker',
    shirtNumber: 99
  },
];

interface Player {
  name: string;
  position: string;
  shirtNumber: number;
}
```

אז בגישה הישנה היינו עושים לולאת for פשוטה

```typescript
function getPlayersNames(players: Player[]) : string[] {
  let playersNames: string[] = [];
  for (let i = 0; i < players.length; i++) {
    const player = players[i];
    playersNames.push(player.name);
  }
  return playersNames;
}
```
אנחנו נשתמש במטודה forEach. 
שיטה זו **תפשט** בצורה דרמטית את הקוד, למרות שלא חשוב רק מספר השרות שבהן הקוד הכתוב - אך הסיבה שאני מתייחס אליה זה בשונה מלולאת הFor, לולאת forEach יכולה גם לרוץ בצורה אסינכורנית. 
מטודה זו היא מאד נוחה, אם רוצים לעשות משהו על כל איבר במערך.

```typescript
function getPlayersNames(players: Player[]) : string[] {
  let playersNames: string[] = [];
  players.forEach((player: Player) => {
    playersNames.push(player.name);
  })
  return playersNames;
}
```

שימו לב, שזו התנהגות מאד נפוצה. אנחנו מגדירים מערך חדש, עוברים על כל האיברים במערך הישן, עושים משהו ומחזירים את המערך החדש.

[אם נסתכל על הPrototype של **map** ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) נגלה שנחסך לנו המימוש של forEach, במקרים האלה.
אז איך היינו כותבים את אותה לוגיקה, רק בשימוש במטודה map?

```javascript
function getPlayersNames(players: Player[]) : string[] {
  return players.map(player => player.name);
}
```

### Filter

בעזרת map נעבור על כל הפריטים במערך ונבצע פעולה מסויימת, אבל מה אם נרצה לקבל רק חלקים מהפריטים במערך? filter עושה בדיוק את זה. לפעמים נרצה לפלטר את הפריטים במערך, ולהחזיר מערך חדש המחזיק פריטים העונים לתנאי מסויים.

בואו נמשיך עם הדוגמא של שחקני הכדורגל. בואו נדמיין שנרצה לקבל רק שחקנים שמשחקים בעמדת מסויימת. תחלה נשתמש במטודה forEach. בדומה לדוגמא הקודמת נצהיר על מערך חדש, נרוץ על הפריטים במערך שקיבלנו ועם הלוגיקיה שלנו ניצור מערך חדש. 



```typescript
function getPlayersPosition(players: Players[], position: string) {
  let playersNames: string[] = [];
  players.forEach((player: Player) => {
    if (player.position === position) {
      playersNames.push(player.name);
    }
  });
  return playersNames;
}
```



אבל, בדומה לProtoype של map, המימוש הזה נחסך לנו עם שימוש בfilter. האיברים אשר יתאימו לתנאי יחזרו בתור מערך חדש. 

```typescript
function getPlayersPosition(players: Players[], position: string) {
  return players.filter(player => player.position === position);
}
```



### שירשור

אז למדנו שלוש מטודות מאד חשובות, אבל הכוח האמיתי הוא בשילוב שלהם. אם נדע לשלב אותם נכון, נייתר את הצורך בלולאות בכלל. תזכורת קלה: אנחנו יכולים להשתמש בלולאות רק שנרצה לעבוד עם מידע שזמין בזיכרון ואנחנו נרצה לעבוד עם מידע שמגיע בצורה אסינכורנית כמו לדוגמא, events.

אז בדוגמא הזאת, נדגים קיצר אפשר לקבל שמות של שחקנים אשר משחקים בעמדה מסויימת. המטודות map וfilter, מושלמות למשימה הזאת. תחילה נסנן את השחקנים המתאימים לעמדה שבחרנו בעזרת filter, ובעזרת המטודה map נקבל את כל השמות של השחקנים שהתאימו לסינון. **תזכורת**: כל אחת מהמטודות מחזירה מערך חדש. 

```typescript
function filteredPlayersNamesByPosition(players: Players[], position: string) {
  return players
      .filter(player => player.position === position)
      .map(player => player.name);
};

```



### לסיום - Observable על קצה המזלג

ניתן להשתמש ב Observables בשביל למדל אירועים, בקשות אסינכרוניות ואנימציות. אפשר לצרוך אותם, לשנות אותם, לשלב אותם בעזרת המטודות שלמדנו בשיעורים הקודמים. רק בעזרת המטודות האלה, אפשר לכתוב אפליקציות אסינכורניות מאד חזקות.   Observable מתנהג בדיוק כמו מערך רק שהוא לא נגיש מיידית מהזכרון. כלומר, האיברים בObserable מגיעים במהלך זמן בצורה אסינכרונית בשונה ממערך, אשר האיברים נמצאים בזיכרון. אני משתמש בסיפריה rx.js בשביל להשתמש בObservable. 

המשך יבוא :)







