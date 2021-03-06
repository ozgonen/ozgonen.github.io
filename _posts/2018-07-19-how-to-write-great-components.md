---
layout: post
title: איך לכתוב קומפוננטות, ברמה הגבוהה ביותר
description: "כולנו יודעים איך לעשות את זה. בערך. איך נכתוב קומפוננטות ברמה הגבוהה ביותר? "
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [Angular]
image: '/images/posts/2018-07-19-how-to-write-great-components.jpg'

---



#### איך לכתוב קומפוננטות, ברמה הגבוהה ביותר

כמעט כולם יודעים מה זה קומפוננטות ״טיפשות״ ו״חכמות״. (או, אם אתם באים מריאקט [Presentational and Container](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)) אנחנו יודעים שצריך להשתמש בעיקר בInput() וב Output. אבל שהאפליקצייה מתחילה להיות ענקית, זה מתחיל להראות קצת קוד ספגטי שאנחנו למדנו לשנוא. 

![spaghetti](/images/gifs/spaghetti.gif)

אנחנו הרי יודעים מהי תבנית פיתוח טובה (Design Pattern) אבל יש לי תחושה שהרבה מפתחים מתבלבלים עם מה חובה ומה הכרחי. האם אני חייב ״לאנוס״ את הקוד בשביל שיתאים לי לתבנית? הרבה מהתבניות האלה, שאנחנו עובדים לפיהם, מתחילות להיות מטושטשות ובסופו של דבר אנחנו משתמשים ביותר קיצורי דרך ממה שאנחנו אמורים להשתמש. 

אחת מהתבניות האלה, היא לפצל את הקומפוננטות שלנו לקומפוננטות טיפשות וחכמות. כל הלוגיקה העסקית (Busniess Logic) צריכה להתנהל בתוך הקומפוננטה החכמה בעוד ששאר הקומפוננטות צריכות להיות כמה שיותר טיפשות ועם מעט מאד לוגיקה. התבנית הזאת היא מעולה - אבל הדבר הקשה ביותר הוא להתמיד בה, בסופו של דבר הרבה מאד מתכנתים זוכרים אותה בראש כרעיון באופן כללי ובסופו של דבר מוותרים על הקונספט הזה יותר ויותר. 

לדוגמא, נניח ואנחנו עובדים לפי התבנית. ובאיזשהו שלב, אנחנו רוצים להעביר מידע מהקומפוננטת אב מידע לנכד של הנכד שלה. זאת חפירה רצינית להתחיל להתמודד עם כל קופננטה ואיך שהיא מעבירה את המידע לקומפוננטה אחריה בהיררכיה, אז אנחנו נהיים עצלנים ופשוט משנים את המידע בצורה ישירה. הרבה פעמים אנחנו גם לוקחים את הגישה של אנגולר1 - משתמשים בסרביס כסינגלטון, שכל קומפונטטה משתמש בו כל פעם שהיא זקוקה לזה. זה הופך את כל הקומפוננטות לחכמות, אבל הבעיה היא שאנחנו לא יכולים לעבוד עם [OnPush](https://angular.io/api/core/ChangeDetectionStrategy), בכל קומפוננטה שמעל הקומפוננטה החכמה.  

אנחנו עלולים למצוא את עצמנו באפליקציה עם אלפי קומפוננטות, הרבה סרביסים וכל אחד תלוי בשני. קשה מאד לעקוב מה תלוי במה ואז מרגישים כמו ״חוש חש״ הבלש בשביל לדבג או פשוט להבין מה הולך באפליקציה. 

![inspectorGadget](/images/gifs/inspectorGadget.gif)

**הפרוייקט הופך להיות קשה לתחזוקה, מה שגורם לנו להתעצבן, אנחנו לא מבינים מה קורה באפליקציה ואז בסופו של דבר אני מוצא את עצמי מקלל ואומר: ״הכל הרבה יותר קל היה עם jQuery״.** 

זה לא אמור להיות ככה.

### הפתרון

אני לא מתיימר לשלום עולמי או לפתור את כל תחלואות העולם. לדעתי, הסיבה שהכל קורה זה פשוט קשה לנו לעבוד לפי התבנית של קומפוננטות חכמות וטיפשות. אנחנו מבצעים אותה בצורה חלקית. אני באמת מאמין שאם נבצע אותה בצורה מושלמת, נוכל לבנות קומפוננטות ברמה הגבוהה ביותר. 

כחלק מהתכנון של בניית רכיב מסויים במערכת, אני עושה את הפעולות הבאות:

1. מחלק את הקומפוננטות לטיפשות וחכמות.

2. משתדל שיהיה לי כמה שיותר קומפוננטות טיפשות וכמה שפחות קומפוננטות חכמות. 
3. מנסה למצוא סיבה מספיק טובה בשביל להפוך קומפוננטה טיפשה לחכמה - כלומר נקודת המוצא שלי היא קומפוננטה טיפשה. ממש מנסה לשכנע את עצמי למה היא חייבת להיות חכמה.

### מה זה באמת אומר, קומפוננטה חכמה וטיפשה?

קומפוננטה טיפשה מתנהגת כמו פונקציה טהורה (Pure Function), כלומר שבהינתן ארגומטים מסויימים היא תמיד תחזיר את אותה תוצאה ותתנהג בדיוק אותו דבר. ניתן לה מידע א׳, יחזור לנו תמיד מידע ב׳. **תמיד.** 

אם הקומפוננטה זקוקה למידע, היא תקבל אותו בעזרת Input. אם היא מפיקה מידע כלשהו, היא תוציא אותו בעזרת Output. היא לא אמורה לשנות (mutate) את ה Input שהיא מקבלת. 

![dumb](/images/gifs/dumb.gif)

קומפוננטה חכמה, היא קומפוננטה שנוגעת בעולם החיצון. יש לה אינטרקציה עם רכיבים אחרים במערכת בין אם היא מקבלת מידע או מפיקה תוצרי לוואי (Side Effects). היא לא תלויה רק ובמידע שהיא מקבל כInput, יכול להיות שהיא תקבל מידע מסרביס, או מהStore, מהLocalStorage, קריאת API ועוד.  מין הסתם, בכיוון השני קומפוננטה חכמה יכולה לעשות שינויים במידע שלא דרך Output. לשנות את המידע שנמצא בLocalStorage, בסרביס ועוד. 

![dumb](/images/gifs/smart.gif)

### איך באמת מחליטים מי תהיה טיפשה ומי חכמה?

יש לי כמה כללי אצבע שעוזרים לי, תוכלו להעזר בהם. 

1. האם אני יכול לחזות מראש מה התפקיד של הקומפננטה? - אם כן, כנראה שהיא יכולה להיות טיפשה. לדוגמא: להציג כפתור אישור וכפתור ביטול. קל לי  לקבוע את המידע שיכנס לקומפוננטה (האם הכפתורים פעילים, האם להציג אותם) והמידע שיחזור (איזה כפתור נלחץ)
2. אם קומפוננטות אחיות הן חכמות, אפשר להפוך אותם לטיפשות -  אפשר בקלות להפוך את הקומפוננטת אב שלהם להיות חכמה, ושהיא תנהל את הלוגיקה. אין צורך בלוגיקה כפולה בכל אחת מהקומפוננטות. 
3. מה שלא יכול להיות טיפש… שיהיה חכם! - הרי אנחנו לא יכולים להפוך את כל הקומפוננטות שלנו להיות טיפשות. לפחות אחת מהם צריכה להיות חכמה. ברוב המקרים, אני בוחר בקומפוננטת אב שתהיה חכמה. היא מנהלת את התקשורת עם העולם החיצוני ומעבירה את המידע לקומפוננטות ילדים שלה. כמובן שלא הגיוני שקומפוננטה אחת תנהל לנו את כל הלוגיקה, אז אפשר לפצל את זה לקומפוננטות למספר קומפוננטות חכמות. וזה בסדר. 

### ״הכל טוב ויפה אבל יש לי קומפוננטה חכמה, וצריכה להעביר מידע לנין של הנין שלה. אני לא אתחיל לכתוב מלא Input.״

אני מבין ללב כל מי שאי פעם חשב את זה. אבל יש לזה פתרונות לזה ועדיין לשמור על התבנית הזאת. אף אחד לא אמר שאי אפשר שבת של קומפוננטה חכמה תהיה חכמה גם היא. חלק מהתקשורת של הקומפוננטה עם העולם החיצון יכולה להיות תקשורת עם הקומפוננטה אב.  

אחרי שהסברתי הרבה על התבנית הזאת, תנו לי להסביר לכם למה

### לכתוב קומפוננטות חכמות וטיפשות =  קומפוננטות ברמה הכי גבוהה שיש

אפשר בקלות לצפות מה קומפננטה טיפשה תעשה, רק לפי ההסתכלות על הinput ועל הoutput שלה.

אפשר לכתוב טסטים בקלות מאד לקומפוננטות טיפשות.

אפשר לעשות לקומפוננטות טיפשות שכתוב (Refactor) בקלות.

קל מאד להבין מה הולך באפליקציה. פשוט מחפשים את הקומפוננטות החכמות והם יובילו את הדרך.

זה נותן ביצועים טובים יותר. באנגולר, אפשר להשתמש באסטרטגיה onPush על הקומפוננטות הטיפשות אבל רק צריך להזהר, אם יש לנו קומפוננטה חכמה בהמשך העץ.



### סיכום

שימוש בתבניות פיתוח יכול להיות כאב ראש רציני. לפעמים אני צריך לאנוס את האפליקציה שלי ולכתוב קיצורי דרך רק בשביל להתאים את עצמי לתבנית. התבנית של קומפוננטות חכמות וטיפשות לא מאפשרת לי להגדיר תלויות באיזה קומפוננטה שאני רוצה ותמיד לנסות לחשוב על מאיפה המידע יגיע מראש. בחלק גדול מהקומפוננטות שלי, אני לא יכול לשנות את הערך של המידע שמגיע לי וזה מאד מבאס.

האם זה שווה את זה? **כן!** 

נכון שעצם זה שאנחנו משתמשים בפריימוורק אמור להפוך את הקוד שלנו ליותר טוב ויותר סקליבלי, אבל ככל שאני מעמיק בזמן ומקפיד יותר ויותר על השימוש בתבנית הזאת, אני מגלה עד כמה היא מצילה אותי. אני מגלה עד כמה שימוש בקומפוננטות כמו שיותר קטנות הופך את חווית הפיתוח (ובמיוחד חווית המשך הפיתוח / תיקוני באגים) לחוויה הרבה יותר טובה ומהירה.

