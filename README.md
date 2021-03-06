[![Build Status](https://travis-ci.org/Eutrepe/ng-ep-intersection-observer.svg?branch=master)](https://travis-ci.org/Eutrepe/ng-ep-intersection-observer)[![codecov](https://codecov.io/gh/Eutrepe/ng-ep-intersection-observer/branch/master/graph/badge.svg?token=8QNK5IT1KH)](https://codecov.io/gh/Eutrepe/ng-ep-intersection-observer)
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release)
![GitHub](https://img.shields.io/github/license/Eutrepe/ng-ep-intersection-observer?style=plastic)


---

# NgxEutrepeIsInViewport

* Directive for Angular 10+ based on Intersection Observer to detect element in viewport.

## Installation

`npm i --save @eutrepe/@eutrepe/ng-ep-intersection-observer`

You also need polyfill

`npm i --save intersection-observer`

and add this in your `src/polyfills.ts` file

`import 'intersection-observer';`

<br />
<br />

# Usage

### 1) Register the `EpIntersectionObserverModule` in your app module.
 > `import { EpIntersectionObserverModule } from '@eutrepe/ng-ep-intersection-observer'`

 ```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';

import { EpIntersectionObserverModule } from '@eutrepe/ng-ep-intersection-observer';

@NgModule({
    declarations: [
      AppComponent
    ],
    imports:      [
        BrowserModule,
        EpIntersectionObserverModule
    ],
     providers: [],
    bootstrap: [ AppComponent ],
    exports: []
})
export class AppModule { }
```
 <br /><br />
 

  ### 2) Use the directive `ep-io`

 If element is visible in viewport the directive add `in-viewport` class to the element or custom class


 #### Basic

 ```html
<section class="section1" ep-io>
  <p>Lorem ipsum, dolor sit amet consectetur adipisicing elit. Harum, facere.</p>
</section>
```

#### Advanced

 ```html
<section class="section1"
    [ep-io]="{threshold: [0, 0.5, 1]}"
    [epIoOnActive]="onActive.bind(this, param1, param2, ...)"
    [epIoOnUnactive]="onUnactive.bind(this, param1, param2, ...)"
    [epIoInvokeOnce]="true"
    [epIoWaitFor]="event"
    [epIoGroupName]="'myGroupName'"
    [epIoActiveClass]="'my-class'"
    [epIoRemoveClassAfterLeaving]="false"
    (epIoViewportChange)="onChanage($event)"
    >
  <p>Lorem ipsum, dolor sit amet consectetur adipisicing elit. Harum, facere.</p>
</section>

<!-- Use .bind(this) for callbacks if you want use scoped variables -->
```


#### Advanced 'Lazy'

If you have multiple directives in the same options you can use global configuration per module or per component using `INTERSECTION_OBSERVE_GLOBAL_CONFIG` in `providers` array. Unfortunately here you can't use `this` or other dependency inside `epIoOnActive` and `epIoOnUnactive` functions but you can use subscription from `NgEpIOStatusService` service and manually control this logic.

 ```html
<section class="section1" 
  ep-io
  (epIoViewportChange)="onChanage($event)">
  <p>Lorem ipsum, dolor sit amet consectetur adipisicing elit. Harum, facere.</p>
</section>
```

```typescript
import { Component, OnInit } from '@angular/core';
import { INTERSECTION_OBSERVE_GLOBAL_CONFIG, EpIntersectionObserverStatusService, IntersectionObserverEvent } from '@eutrepe/ng-ep-intersection-observer';

@Component({
  selector: 'app-io-event-example',
  templateUrl: './io-event-example.component.html',
  styleUrls: ['./io-event-example.component.scss'],
  providers: [
    {
      provide: INTERSECTION_OBSERVE_GLOBAL_CONFIG, useValue: {
        intersectionConfig: {
          rootMargin: '-20% 0px -20% 0px',
          threshold: [0, 0.5, 1]
        },
        epIoGroupName: 'myGroupName',
        epIoWaitFor: 'event',
        epIoOnActive: () => {console.log('Yoy can\'t use here "this" operator')}
        epIoOnUnactive: () => {console.log('Yoy can\'t use here "this" operator')}
        epIoInvokeOnce: true,
        epIoActiveClass: 'my-class',
        epIoRemoveClassAfterLeaving: false
      }
    }
  ]
})
export class IoEventExampleComponent implements OnInit {

  constructor(private ngEpIOStatusService: EpIntersectionObserverStatusService) { }

  ngOnInit(): void {  

    this.ngEpIOStatusService.get().subscribe((event: IntersectionObserverEvent) => {
      console.log(event.el);
      console.log(event.status);
    });
  }
}
```


If you want run directive logic after event (for example load all images, videos etc.) you should set property `epIoWaitFor` to `event` and use `NgEpIOService`.
Also if you have multiple directive with multiple events you need add `epIoGroupName` to each directive. If you have multiple directive but all waiting for the same event you can omit this property and run `send` function from `NgEpIOService` invoke without parameter

```typescript
import { Component, OnInit } from '@angular/core';
import { EpIntersectionObserverWatchService, INTERSECTION_OBSERVE_GLOBAL_CONFIG } from '@eutrepe/ng-ep-intersection-observer';

@Component({
  selector: 'app-io-event-example',
  templateUrl: './io-event-example.component.html',
  styleUrls: ['./io-event-example.component.scss'],

  providers: [
    {
      provide: INTERSECTION_OBSERVE_GLOBAL_CONFIG, useValue: {
        intersectionConfig: {
          rootMargin: '-20% 0px -20% 0px',
        },
        epIoGroupName: 'myGroupName',
        epIoWaitFor: 'event'
      }
    }
  ]
})
export class IoEventExampleComponent implements OnInit {

  constructor(private ngEpIOWatchService: EpIntersectionObserverWatchService) { }

  ngOnInit(): void {
    // Intersection Observer will start work after 5 sec. *****
    setTimeout(() => {
      this.ngEpIOWatchService.start('myGroupName');
    }, 5000);
  }
}

```
<br />
<br />

# API

#### Directive:

| Input                               | Type                        | Required                                              | Description                                                            |
| ----------------------------------- | --------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------- |
| [ep-io]                             | IIntersectionObserveConfig  | **YES** , default Intersection Observe configuration* | Run the plugin with user configuration or default configuration        |
| [epIoInvokeOnce]                    | boolean                     | Optional, default: false                              | If true directive invoke just once                                     |
| [epIoOnActive]                      | Function                    | Optional, default: null                               | The function is invoked when element is in viewport                    |
| [epIoOnUnactive]                    | Function                    | Optional, default: null                               | The function is invoked when element is out viewport                   |
| [epIoWaitFor]                       | string                      | Optional, default: `none`                             | Inform directive when invoke self logic. `none` - immediately, `media` - after loading all images, videos, audios INSIDE every SINGLE directive, `event` - waiting for event from service                    |
| [epIoGroupName]                     | string                      | Optional, default: `` (empty string)                  | After send event to invoke directive logic, all directives with matched group name will start. This property set group name for directive                     |
| [epIoActiveClass]                   | string                      | Optional, default: 'in-viewport'                      | Custom class for visible element                                       |
| [epIoRemoveClassAfterLeaving]       | boolean                     | Optional, default: true                               | If set true directive remove CSS class after leaving viewport                                      |

*Intersection Observe: [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

<br />


#### Services:

##### EpIntersectionObserverWatchService:

This function inform directive when start self logic if property `epIoWaitFor` is set to `event`

```typescript
get(): Observable<string>
``` 
- Observer for event. Used inside directive
<br />
```typescript 
start(groupName?: string = ''): void
``` 
- When you run this function directive will start self logic. In params you can send groupName what decide which directives can run. Property `epIoGroupName` in directive must set for the same value


##### EpIntersectionObserverStatusService:

```typescript
get(): Observable<IntersectionObserverEvent>
``` 
- Observer for event

```typescript 
send(status: IntersectionObserverEvent): void
``` 
- Sending event when visibility status was changed. Used inside directive

<br />
<br />

# types

```typescript 
export type IntersectionObserveConfig = {
  root?: HTMLElement;
  rootMargin?: string;
  threshold?: Array<number>;
}


export type IntersectionObserverEvent = {
  el: HTMLElement;
  status: boolean;
}


export type IntersectionObserveGlobalConfig = {
  intersectionConfig?: IntersectionObserveConfig;
  epIoOnActive?: (...args: any[] | null) => void;
  epIoOnUnactive?: (...args: any[] | null) => void;
  epIoInvokeOnce?: boolean;
  epIoWaitFor?: string;
  epIoActiveClass?: string;
  epIoGroupName?: string;
  epIoRemoveClassAfterLeaving?: boolean;
}
``` 


<br />

### Intersection Observe default config:

```typescript
const defaultConfig: IntersectionObserveConfig = {
    root: null,
    rootMargin: '0px 0px 0px 0px',
    threshold: [0.0]
  }
```

