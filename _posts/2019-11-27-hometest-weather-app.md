---
layout: post
title: פתרון מבחן בית מס׳ 2 - אפליקציית מזג אויר
description: "בואו מעביר מקומפוננטה אחות וישר נציג את המידע"
modified: 2016-04-12T15:27:45-04:00
comments: true
author: oz
tags: [HomeTests]
image: '/images/posts/2019-11-27-hometest-weather-app.jpg'

---



לאחר תקופת צינון, אני ממשיך עם הצגת פתרונות מטלות בית.

אם פספסתם את המאמר הקודם, אפשר למצוא אותו [כאן](http://www.ozgonen.co.il/2018/08/02/hometest-controll-ball/).

#### מבחן בית מס׳ 2 - אפליקציית מזג אוויר

![juggle](/images/gifs/weather.gif)

**רמת קושי:** מתכנת מתחיל

**תיאור**: צרו אפליקציה שמציגה את מזג האוויר.

השתמשו בAPI של [AccuWeather](https://developer.accuweather.com/) - בשביל לבצע [AutoComplete](https://developer.accuweather.com/accuweather-locations-api/apis/get/locations/v1/cities/autocomplete), [Current Weather](https://developer.accuweather.com/accuweather-current-conditions-api/apis/get/currentconditions/v1/%7BlocationKey%7D), [5 Day forecast](https://developer.accuweather.com/accuweather-forecast-api/apis/get/forecasts/v1/daily/5day/%7BlocationKey%7D).

**דרישות:** צרו 2 עמודים שאפשר לנווט בינהם:

1. עמוד בית - יציג שדה חיפוש שבו אפשר לחפש מזג אוויר לפי מיקום. הקומופננטה תהיה auto completed. מתחתיו, יהיה מזג האוויר לחמשת הימים של המיקום שהוקלד. המיקום יוכל להישמר/להמחק מ״מועדפים״, כאשר תהיה אינדיקציה אם הוא נמצא שם. המיקום הדיפולטיבי הוא ת״א.  (בונוס, להשתמש [בgeoloaction api](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API))
   * בונוס: להוסיף טוגל של dark/light theme
   * בונוס: להוסיף טוגל של Celsius/Fahrenheit
2. עמוד מועדפים - יציג את רשימת המועדפים. כל מיקום מועדף יציג את השם שלו ואת המזג האוויר הנוכחי. לחיצה עליו, תנווט לעמוד הבית עם המיקום הנבחר.

* כל החיפושים יהיו באנגלית.
* חובה להשתמש בstate managment.

**[אפשר לראות את הקוד כאן](https://qgnirjkrb.github.stackblitz.io)**

### ביצוע

נתחיל מליצור פרוייקט חדש, אני אוהב לעבוד עם הקונפיגורציות האלה:

```js
ng g weatherApp --style=scss --skipTests=true --routing=true
```



שימו לב, שהפעלתי את הדגל של Routing, בשביל שיווצר routing NgModule.

אחרי שסיימנו ליצור את הפרוייקט, ניצור 2 קומפוננטות עבור כל אחד מהעמודים, ונתחיל לעצב את עמוד הבית.

נרצה לבנות Header, שיכיל את הניווט של האפליקציה, את החלפת המצבים והטמפרטורה.

#### ניווט

 נתחיל בלהוסיף את הRoutes לקובץ app-routing:

<script src="https://gist.github.com/ozgonen/92b2034d5068318287e6324ee656f4fd.js"></script>

ונמשיך בבניה של הניווט שלהם:

```html
  <nav>
    <a routerLink="/" routerLinkActive="active"
       [routerLinkActiveOptions]="{exact:true}">index</a>
    <a routerLink="/fav" routerLinkActive="active">fav</a>
  </nav>
```

אני ארצה להוסיף איזשהי אידנקציה שהעמוד באמת נבחר, ובגלל זה הוספתי את  RouterLinkActive, שיוסיף את הקלאס active  במידה ואני נמצא בתוך העמוד. שימו לב, שאני צריך להגדיר לו לexact:true - שלא יתבלבל עם הroute השני.

#### מצב חשוך (dark) / מואר (light)

בעזרת css variables, ותיכנון מוקדם, אפשר להשיג את הבונוס הזה יחסית בקלות.

תחילה ניצור משתנים לכל הצבעים שנרצה להשתמש בהם, בתוך הקובץ יעודי, theme.scss:

—GIST

מה שעומד מאחורי זה, שילוב של SCSS Maps ומשתנים של CSS. אפשר לקרוא עוד על זה [כאן](https://dev.to/adamaso/angular-6-dynamic-themes-without-a-library-2e9c).

מכאן, נשאר רק לבנות כפתור, את הלחיצה ולהחליף את הצבע של המשתנה.



```typescript
  mode = Mode.Light;

  switchMode(mode) {
    if (mode === Mode.Light) {
      document.querySelector('body').style.setProperty('--bg-color', '#272727');
      document.querySelector('body').style.setProperty('--text-color', '#f8fafb');
      this.mode = Mode.Dark;
    } else {
      document.querySelector('body').style.setProperty('--bg-color', '#f8fafb');
      document.querySelector('body').style.setProperty('--text-color', '#272727');
      this.mode = Mode.Light;
    }
```

שימו לב, שהיה אפשר ליצור Class עזרה, שהיה יכול לקרוא את הערכים והמשתנים מקובץ הSCSS, במקום לכתוב את הצבעים ידנית. אפשר לקרוא על זה [כאן](https://stackoverflow.com/questions/40418804/access-sass-values-colors-from-variables-scss-in-typescript-angular2-ionic2).



### עמוד הבית

דבר ראשון שנרצה, כשהקומפוננטה עולה, זה להשיג את המיקום הגאוגרפי של המשתמש.



```typescript
  ngOnInit() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition((position) => {
        const {latitude, longitude} = position.coords;
        this.appService.getGeoPosition(latitude, longitude).subscribe((data: GeoPositionRes) => {
          this.handleInitPosition(data);
        });
      });
    } else {
      this.appService.getGeoPosition(DEFAULT_LAT, DEFAULT_LNG).subscribe((data: GeoPositionRes) => {
        this.handleInitPosition(data);
      });
    }
  }
```

נגיד באופן דיפולטיבי את הקורדיננטות של ת״א, שהשגתי [כאן](https://www.latlong.net/place/tel-aviv-yafo-israel-7174.html) ובhook של אנגולר ngOnInit,  השתמשתי בapi geolocation להקפיץ למשתמש את ההודעה אם הוא מרשה לאפליקציה לקבל מידע הגאוגרפי שלו. במידה וכן, אני דורס את המידע הדיפולטיבי שהגדרתי.

הדבר הבא, הוא להתחיל להשתמש בקריאות API. נתחיל בלהרשם ל[accuweather](https://developer.accuweather.com/) ונתחבר.

ארצה לבנות service שינהל את כל הקריאות API, הקריאה הראשונה היא תהיה [קריאה בשביל להמיר את המידע של הקורדינטות](https://developer.accuweather.com/accuweather-locations-api/apis/get/locations/v1/cities/geoposition/search) למידע שאוכל להשתמש, locationKey - בשביל Current Conditions. אשתמש בפונקציה גנרית, שתבנה לי את ה HttpParams, ותוסיף את ה API_KEY במקום אחד.

app.service.ts

```typescript

  getRequest(url, q?) {
    const params = new HttpParams({fromObject: {apikey: API_KEY, q}});
    return this.http.get(url, {params});

  }

  getGeoPosition(lat: number, lng: number): Observable<any> {
    const url = `http://dataservice.accuweather.com/locations/v1/cities/geoposition/search`;
    return this.getRequest(url, `${lat},${lng}`);
  }

  getAutoComplete(key: string): Observable<any> {
    const url = `http://dataservice.accuweather.com/locations/v1/cities/autocomplete`;
    return this.getRequest(url, `${key}`);
  }

  get5DaysOfForecasts(key: string): Observable<any> {
    const url = `http://dataservice.accuweather.com/forecasts/v1/daily/5day/${key}`;
    return this.getRequest(url);
  }


  getCurrentConditions(key: string): Observable<any> {
    const url = `http://dataservice.accuweather.com/currentconditions/v1/${key}`;
    return this.getRequest(url);
  }
```

השלב הבא, הוא לקבל את התחזית הרלוונטית. ארצה גם לבנות String, שיכיל את השם של העיר שאותה אני מחפש.

index.component.ts

```typescript
  private handleInitPosition(geoPositionRes: GeoPositionRes) {
    this.cityName = `${geoPositionRes.ParentCity.EnglishName},${geoPositionRes.Country.EnglishName}`;
    this.appService.get5DaysOfForecasts(geoPositionRes.Key).subscribe((fiveDaysForecastData: FiveDaysForecast) => {
      this.headLine = fiveDaysForecastData.Headline.Text;
      this.forecasts = fiveDaysForecastData.DailyForecasts;
    });
  }
```

נציג את המידע בצורה יפה, נציג אייקון ונפרסר את התאריך עם pipe של אנגולר.

נוסיף input עבור הautocomplete, ונציג גם את התוצאות שחוזרות מהשרת. נוסיף גם כפתור של הוספה / הסרה ממועדפים.

index.component.html

```html
<input type="text" class="autoComplete"
       [ngModel]="autoCompleteValue"
       (input)="autoCompleteInput.next($event.target.value)">

<div class="autocomplete-suggestions" *ngIf="autoCompletedSuggestions">
  <div class="suggestion" *ngFor="let suggestion of autoCompletedSuggestions" (click)="selectSuggestion(suggestion)">
    {{suggestion.LocalizedName}}, {{suggestion.Country.LocalizedName}}
  </div>
</div>


<div class="title">
  <div class="name">{{cityName}}</div>
  <div class="headLine">{{headLine}}</div>
</div>

<div class="fav" *ngIf="cityName" (click)="toggleFavorites()">{{favState}} Favorites</div>


<div class="forecasts">
  <div class="forecast" *ngFor="let forecast of forecasts;">
    <div class="date">{{forecast.Date | date: 'fullDate'}}</div>
    <div class="phrase"> {{forecast.Day.IconPhrase}}</div>
    <div class="tempature"> {{forecast.Temperature.Minimum.Value}}{{forecast.Temperature.Minimum.Unit}} - {{forecast.Temperature.Maximum.Value}}{{forecast.Temperature.Maximum.Unit}}</div>
    <img src="{{forecast.Day.Icon | accuweatherIcon }}"/>
  </div>
</div>

```

שימו לב שאני משתמש ב Pipe בשביל לשלוט בתמונה שתוצג לפי המזג אוויר - אם המספר של האייקון הוא קטן מ10, צריך להוסיף את התו ״0״ בשביל להציג את התמונה.

```typescript
import { Pipe, PipeTransform } from '@angular/core';

export const IMG_URL = `https://developer.accuweather.com/sites/default/files`;
@Pipe({
  name: 'accuweatherIcon'
})
export class AccuweatherIconPipe implements PipeTransform {


  transform(value: any): any {
    if (value < 10) {
      value = `0${value}`;
    }
    return `${IMG_URL}/${value}-s.png`;
  }

}

```



ניצור Subject שינהל את הקלדת התווים ונשתמש באופרטורים debounceTime וswitchMap בשביל לוודא שלא נציף את השרת בקריאות מיותרות.

בכל לחיצה על תוצאה, נקח את המידע ונבצע קריאה נוספת עם המידע החדש.

```typescript
   this.autoCompleteInput
      .pipe(
        filter((data: string) => data.length > 0),
        takeUntil(this.ngUnSubscribe),
        debounceTime(300),
        switchMap((data: string) => {
          return this.appService.getAutoComplete(data);
        })
      )
      .subscribe((suggestions: AutoCompleteSuggestions[]) => {
        this.autoCompletedSuggestions = suggestions;
      });
  }

  selectSuggestion(suggestion: AutoCompleteSuggestions) {
    this.cityName          = `${suggestion.LocalizedName},${suggestion.Country.LocalizedName}`;
    this.autoCompleteValue = this.cityName;
    this.getFiveDays(suggestion.Key);
    this.autoCompletedSuggestions = null;
  }
```

### מועדפים

בתרגיל הזה, ביקשו מאיתנו להשתמש ב state managment. אני חושב שבמקרה הזה ngrx/mobx זה ממש overkill, והייתי מציע להשתמש במשהו פשוט יותר, לדוגמא [Observable Store](https://blog.codewithdan.com/simplifying-front-end-state-management-with-observable-store/). לא הייתי משתמש בספרייה הזאת בproducation, אלא יוצר [סרביס כזה בעצמי](https://dev.to/steveblue/redux-with-observable-stores-in-angular-1b3b) - או באמת משתמש בngrx/mobx.

אני אתקין את החבילה מnpm

```
npm install @codewithdan/observable-store
```

 ואצור סרביס חדש, בשם weather.service.ts

<script src="https://gist.github.com/ozgonen/f022994b9142ead0b09749ab485c60d6.js"></script>

בשביל לדעת אם העיר שלנו נמצאת במועדפים, אצור פונקציית עזר ואשתמש בה כל פעם בשביל לאתחל את המשתנה favState

Index.component.ts

```typescript
private getFavState(Key: string) {
  const storeState = this.weatherService.get();
  return storeState[Key] ? REMOVE_FAV : ADD_FAV;
}

selectSuggestion(suggestion: AutoCompleteSuggestions) {
  this.favState          = this.getFavState(suggestion.Key);
  this.cityName          = `${suggestion.LocalizedName},${suggestion.Country.LocalizedName}`;
  this.autoCompleteValue = this.cityName;
  this.getFiveDays(suggestion.Key);
  this.autoCompletedSuggestions = null;
}


private handleInitPosition(geoPositionRes: GeoPositionRes) {
  this.favState    = this.getFavState(geoPositionRes.Key);
  this.cityName    = `${geoPositionRes.ParentCity.EnglishName},${geoPositionRes.Country.EnglishName}`;
  this.getFiveDays(geoPositionRes.Key);
}
```

כמובן, אוסיף את המטודה שאחראית על להוסיף / להסיר את העיר מהמעודפים

```typescript
toggleFavorites() {
  const faveState = this.getFavState(this.selectedKey);
  const selectedCity = {
    key: this.selectedKey,
    cityName: this.cityName
  };
  if (faveState === ADD_FAV) {
    this.weatherService.add(selectedCity);
  } else {
    this.weatherService.remove(selectedCity);
  }

  this.favState = faveState === ADD_FAV ? REMOVE_FAV : ADD_FAV;
}
```

נאזין לשינויים של הsubject, ונוסיף את האובייקט למערך שנדפיס בטמפלט

<script src="https://gist.github.com/ozgonen/1b1c8c2e3de8d333c72a6791beeee824.js"></script>



### סיכום

עשינו תרגיל שמצריך הכרה של דברים בסיסים כמו Routing, API, RxJS, State Managment. התרגיל הזה בדרגת קושי קלה ומתאים בעיקר למפתחים מתחילים.



אפשר לראות את הקוד [כאן](https://stackblitz.com/github/qgnirjkrb) ואת התוצאה [כאן](https://qgnirjkrb.github.stackblitz.io).

