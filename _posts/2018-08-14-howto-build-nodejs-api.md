---
layout: post
title: איך לבנות API של אלופים עם NodeJS
description: "עם קצת כלים מגניבים, אפשר לבנות שרת חזק"
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [NodeJS]
image: '/images/posts/2018-02-02-hometest-controll-ball.jpg'

---

#### הקדמה

במאמר הזה, אדגים איך אפשר להשתמש בידע שלנו ב JavaScript וTypeScript בשביל לבנות שרת. אני הולך להשתמש בframework שנקראת Hapi.js בשביל ליצור API ואינטגרציה עם MongoDB בתור מסד נתונים. Hapi.js יגיש לנו בצורה סטטית גם את הקבצים של האתר, ככה שנוכל לבצע אינטגרציה עם האפליקצייה front end שלנו. 

כמו תמיד, כל הקוד נמצא כאן:

https://github.com/ozgonen/NodeJS-Hapi



#### בואו נתחיל!

קודם כל, אם אין לכם [NodeJS](https://nodejs.org/en/) או אם אין לכם [MongoDB](https://docs.mongodb.com/manual/administration/install-community/) הגיע הזמן להתקין. 



