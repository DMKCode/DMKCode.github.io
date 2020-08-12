---
title: Simple Timer using Angular 2 and RxJS - part 2
date: 2016-09-15 16:22:56
tags: ['Angular2', 'RxJS']
categories: ['Angular2', 'RxJS']
---

> This is Part 2 for creating a 'Simple Timer using Angular 2 & RxJS'.


In this post, I'm going to extend the simple timer we created in [part 1](https://dmkcode.com/2016/08/simple-timer-using-angular-2-and-rxjs/) by adding another component which will start, pause and stop the timer.

We will start by grabbing the source from the previous part on my [Github](https://github.com/DMKCode/timer_angular2_rxjs).
We will pick it up from there.

## The Service
The service is introduced to facilitate communication between the components. There are many ways to do this as describe in the [Angular Docs](https://angular.io/docs/ts/latest/cookbook/component-communication.html) and here, the service way was chosen for communication.

The following code makes up our timer service.

{% gist 5e0f3371981735e40da6accf918defb7 timer.service.ts %}

We start off by importing Injectable and EventEmitter from @angular/core. We use Injectable to annotate our class to make it injectable into our buttons component which will be shown shortly. We use EventEmitter to emit custom events.

``` typescript
import { Injectable, EventEmitter } from '@angular/core';

@Injectable()
export class TimerService {
    .....
}
```
In the class we create the variables to store booleans for indicating the current state of the timer when a particular button is pressed i.e. play, pause, stop. We also create the playPauseStop$ variable as the EventEmitter which will emit the custom event. 

``` typescript
    private play: boolean = false;
    private pause: boolean = false;
    private stop: boolean = true;
    public playPauseStop$ = new EventEmitter();
```

Next we have the functions, playTimer(), pauseTimer() and stopTimer(). The functions are quite similar and simple. They simply set the play, pause and stop variables and then emit at object containing the relevant action as key value pair.

``` typescript
    public playTimer() {
        this.play = true;
        this.pause = false;
        this.stop = false;

        this.playPauseStop$.emit({
            play: this.play
        });
    }
    public pauseTimer() {
        this.play = false;
        this.pause = true;
        this.stop = false;

        this.playPauseStop$.emit({
            pause: this.pause
        });
    }
    public stopTimer() {
        this.play = false;
        this.pause = false;
        this.stop = true;

        this.playPauseStop$.emit({
            stop: this.stop
        });
    }
```

And to make our service available for dependency injection, we import in the app module and add it to the providers array.

``` typescript
.....
import { TimerService } from '../services/timer.service';
@NgModule({
    ...,
    providers: [ TimerService ],
    ...
})
export class AppModule {
}

```

## The Buttons Component 

The following code makes up our buttons component.

{% gist d1364191643e1bf13860485705c49f4b buttons.component.ts %}

The buttons component is created by following the same procedure as with the timer component in part 1 with a couple of slight deferences though.

We start off by importing Component, OnInit, OnDestroy from @angular/core. We also import our service which as mentioned earlier, we will use for communication with the timer.

``` typescript
    import { Component, OnInit, OnDestroy } from '@angular/core';
    import { TimerService } from '../services/timer.service';
```

We implement the OnInit and OnDestroy lifecycle hooks.
The variable playPauseStopUnsubscribe is declared to store the subcription to the timer service custom EventEmitter and the variable play to store the state of play.
We then Inject, the timer service into the class using the constructor and storing it in the variable timerService.

``` typescript
export class ButtonsComponent implements OnInit, OnDestroy {

    private playPauseStopUnsubscribe: any;
    private play: boolean;

    constructor(private timerService: TimerService) {
    }
.....
}
```

In the ngOnInit() function, we suscribe the to the timer service EventEmitter - playPauseStop$ and pass a callback to execute everytime an event is emitted. The callback invokes the setPlay() passing in the object from the EventEmitter. In the ngOnDestroy() function, we call the unsubscribe function on the playPauseStopUnsubscribe to unsubscribe from timer service EventEmitter.

The setPlay() function simply checks if the object has play on it and if so, set the play variable in the component.

The playTimer() function is called when the play button is pressed which calls the playTimer method in the service.  
The pauseTimer() function is called when the pause button is pressed which calls the pauseTimer method in the service.  
The stopTimer() function is called when the stop button is pressed which calls the stopTimer method in the service.  

``` typescript
    ngOnInit() {
        this.playPauseStopUnsubscribe = this.timerService.playPauseStop$.subscribe((res: any) => this.setPlay(res));       
    }

    ngOnDestroy() {
        this.playPauseStopUnsubscribe.unsubscribe();
    }

    private setPlay(res: any) {
        (res.play) ? this.play = true : this.play = false;
    }

    playTimer() {
        this.timerService.playTimer();
    }

    pauseTimer() {
        this.timerService.pauseTimer();
    }

    stopTimer() {
        this.timerService.stopTimer();
    }
```

We then add the buttons tag to 'app.component.html'
``` html app.component.html
<buttons></buttons>
```
and import, and add to the declarations array in 'app.module.ts'

``` typescript
.....
import { ButtonsComponent } from "./buttons.component";
@NgModule({
    ...,
    declarations: [
        ...
        TimerComponent,
        ...   
    ]
    ...
})
export class AppModule {
}
```

## Timer Component Changes

Finally we make some changes to the timer component to subscribe to use the service to communication.
Comments in the code below describe the changes.

``` typescript
....
// import the timer service for communication
import { TimerService } from '../services/timer.service';


....
export class TimerComponent implements OnInit, OnDestroy {
    // variable to subscribe to the timer service EventEmitter
    private playPauseStopUnsubscribe: any;
    ....
    // start variable to tell the timer when to start
    start = 0;
    ...
    // inject the timer service
    constructor(private timerService: TimerService) {        
    }
    ngOnInit() {
        // subscribe to the playPauseStop$ EventEmitter in the timer service 
        this.playPauseStopUnsubscribe = this.timerService.playPauseStop$.subscribe((res: any) => this.playPauseStop(res));
    }
    ngOnDestroy() {
        // unsubscribe from the EventEmitter when the component is destroyed
        this.playPauseStopUnsubscribe.unsubscribe();;
    }

    private playPauseStop(res: any) {
        // check the object passed by the EventEmitter 
        // and call relevant method based on the action
        if(res.play) {
            this.startTimer();
        } else if(res.pause) {
            this.pauseTimer();
        } else if (res.stop) {
            this.stopTimer();
        }
    }

    private startTimer() {
        ....
        this.sub = timer.subscribe(
            t => {
                // use the start variable to begin timer
                this.ticks = this.start + t;
                ....
            }
        );
    }

    // set the start variable and unsubscribe from the timer observable
    // will be used to start timer
    private pauseTimer() {
        this.start = ++this.ticks;
        if (this.sub) this.sub.unsubscribe();
    }
    //re-set all variables and unsubscribe from the timer observable
    private stopTimer() {
        this.start = 0;
        this.ticks = 0;
        this.minutesDisplay = 0;
        this.hoursDisplay = 0;
        this.secondsDisplay = 0;
        if (this.sub) this.sub.unsubscribe();
    }
    ....
}
```
## Run It
To check out the new timer in action, just run the command

``` bash
$ npm install
$ npm run start
```

Go to http://localhost:8080 or http://localhost:8080/webpack-dev-server/ to see the timer!

{% asset_img Simple_TImer_Part2.png 100% simple timer %}
<br>
The complete source code can be found on my [Github](https://github.com/DMKCode/timer_angular2_rxjs-2). 

Thanks for reading!
