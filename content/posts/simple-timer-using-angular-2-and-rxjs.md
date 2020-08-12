---
title: Simple Timer using Angular 2 and RxJS - part 1
date: 2016-08-15 17:25:53
tags: ['Angular2', 'RxJS']
categories: ['Angular2', 'RxJS']
---

> This is Part 1 for creating a 'Simple Timer using Angular 2 & RxJS'.

There are many ways to create a timer (counting up) using javascript and in this post, I would like to show you how to implement one as a component using the Angular2 framework and RxJS library. To find out more about Angular2, visit the official documentation [here](https://angular.io/) and for more RxJS, visit the official documentation [here](http://reactivex.io/rxjs/).

The begin with, we will look at the structure of the project and then walk through the steps for how to create the component. Lets begin!

## Project Structure
```
.
├── node_modules/
├── package.json
├── src
│   └── app
│       ├── assets/
│       ├── index.html
│       └── scripts
│           ├── bootstrap.ts
│           ├── components/
|           |   ├── app.component.ts
|           |   ├── app.component.html
|           |   └── app.module.ts
│           └── vendor.ts
├── tsconfig.json
├── typings/
├── typings.json
└── webpack
    └── webpack.dev.js
```
The structure is organised to be as simple as possible. You can find the skeleton for this project structure on my [Github](https://github.com/DMKCode/dmkcode_angular2_webpack_skeleton).

### Project Root

The project root contains the following files.
 * package.json: contains the list of dependencies and scripts we could run from the command line.
 * tsconfig.json: contains TypeScript configuration.
 * typings.json: conatins the TypeScript definition files.
 * node_modules: directory where the dependencies will reside.
 * src: directory where the application logic resides. Detailed description is provided below.
 * webpack: directory where webpack configuration will reside. [Webpack](https://webpack.github.io/) is the module bundler we will use for this project.
 * typings: directory where the TypeScript definition files will reside.

### src/app

Here is a description of whats contained in the 'src/app' directory
 * vendor.ts: contains imports for all vendor specific imports.
 * bootstrap.ts: used to launch the application by bootstrapping the app module.
 * index.html: the main html file.
 * scripts/components/app.component.ts: the app root component.
 * scripts/components/app.component.html: the template for app root component.
 * scripts/components/app.module.ts: the app module. Angular provides 'NgModule' to help organize an application into cohesive blocks of functionality. Read more [here](https://angular.io/docs/ts/latest/guide/ngmodule.html). 

## The Timer

Ok now lets write some code! 

Let's begin by creating an empty file named 'timer.component.ts' in 'src/app/components/' directory. This will contain the application logic for the timer.

The following code makes up our timer component.

{% gist 4ef717db4f7a85c984be93d156226d3c timer.component.ts %}

#### whats going on?
At the top we first import 'Component' and 'OnInit' from 'angular/core' and, 'Observable' and 'Subscription' from 'rxjs/Rx'.
``` typeScript
import { Component, OnInit } from '@angular/core';
import { Observable, Subscription } from 'rxjs/Rx';
```
{% blockquote %}
The Component decorator allows you to mark a class as an Angular component and provide additional metadata that determines how the component should be processed, instantiated and used at runtime. -- [Angular Doc](https://angular.io/docs/ts/latest/api/core/index/Component-decorator.html)
<br>
The [OnInit](https://angular.io/docs/ts/latest/api/core/index/OnInit-class.html) lifecycle hook provides and ngOnInit() function which is called after all data-bound properties of the component are initialised.
<br>
Observable and Subscription are part of the Reactive Extensions for JavaScript (RxJS) which is a reactive streams library which enables us to deal with asynchronous data streams.
{% endblockquote %} 

When then provide component metadata.
``` typeScript
@Component({
    selector: 'timer',
    template: ` 
        <h1>
            {{hoursDisplay ? hoursDisplay : '00'}} : {{(minutesDisplay) && (minutesDisplay <= 59) ? minutesDisplay : '00'}} : {{(secondsDisplay) && (secondsDisplay <= 59) ? secondsDisplay : '00'}} <br/>
        </h1>
    `,
    styles: [ `
        h1 {
            color: #57acec;
            margin-top: 24px; 
            text-align: center;   
        }    
    `]
})
```
This essentially means the component will be placed in the html tag with the name 'timer'.
We provide the template as a string, displaying the hours, minutes and seconds. We could also provide this template as a separate file as we did with the app component.
And finally, we provide some styles to make the display look a little pretty.

Next up is the actual timer logic.
``` typeScript
    export class TimerComponent implements OnInit {
```
We make sure our component implements the OnInit lifecycle hook.

``` typeScript
ticks = 0;  
minutesDisplay: number = 0;
hoursDisplay: number = 0;
secondsDisplay: number = 0;

sub: Subscription;
``` 
We then declare our variables to store the values   

``` typeScript
ngOnInit() {
    this.startTimer();
}
```
We create the function ngOnInit() which will be called when the component is initialised.
In this function, we call the startTimer() function explained below.

``` typeScript
private startTimer() {
    let timer = Observable.timer(1, 1000);
    this.sub = timer.subscribe(
        t => {
            this.ticks = t;
            
            this.secondsDisplay = this.getSeconds(this.ticks);
            this.minutesDisplay = this.getMinutes(this.ticks);
            this.hoursDisplay = this.getHours(this.ticks);
        }
    );
}
```
In the startTimer() function, we create an observable sequence that produces a value after due time has elapsed and then each period. [Rx.Observable.timer(dueTime, [period], [scheduler])](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/timer.md)
We then subscribe to the observable and every time a value is produced, i.e. 't', we store in the variable 'ticks'. We then get the seconds, minutes and hours (functions explained below) and store them in the respective variables.

```typeScript
private getSeconds(ticks: number) {
    return this.pad(ticks % 60);
}

private getMinutes(ticks: number) {
    eturn this.pad((Math.floor(ticks / 60)) % 60);
}

private getHours(ticks: number) {
    return this.pad(Math.floor((ticks / 60) / 60));
}

private pad(digit: any) { 
    return digit <= 9 ? '0' + digit : digit;
}
```

The getSeconds() function simply gets the remainder of the value, i.e. the ticks from the observable, divided by 60. This way the seconds displayed will never exceed 60 as expected. We then append a '0' at the beginning if the value is below 10 by passing the result to the pad function.

The getMinutes() function first divides the value, i.e. the ticks from the observable, by 60 and the gets the remainder of result divided by 60. This way the minutes displayed will never exceed 60 as expected. We then append a '0' at the beginning if the value is below 10 by passing the result to the pad function.

The getHours() function simply divides the value, i.e. the ticks from the observable, divided by 60 twice. We then append a '0' at the beginning if the value is below 10 by passing the result to the pad function.

AND finally, don't forgot add the timer to 'app.component.html'
``` html app.component.html
<timer></timer>
```

## Run It
To check out the timer in action, just run the command

``` bash
$ npm install
$ npm run start
```

Go to http://localhost:8080 or http://localhost:8080/webpack-dev-server/ to see the timer!

{% asset_img Simple_TImer_Part1.png 100% simple timer %}
<br>
The complete source code can be found on my [Github](https://github.com/DMKCode/timer_angular2_rxjs).

In the next part, I will add another component which will start, pause and stop the timer. 

Thanks for reading!

