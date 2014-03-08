---
layout: post
title: "JavaScript Game Development - Asynchronous Execution Loop"
published: true
---

Obviously one of the core components of any game is the _game loop_. The
central piece of code that is responsible for balancing the execution of
game logic and drawing operations.

In its simplest naive form you could express it as something akin to the
following

```javascript
class Game {
   constructor() {
      // Holds a timestamp indicating when the last tick occurred      .
      // Initially set to null, indicating that no tick has taken place.
      this.time = null;
   }

   step(deltaTime) {
      // ...
   }

   draw(deltaTime) {
      // ...
   }

   run() {
      while(true) {
         var time = Date.now();

         // On the first tick delta time should be 0.
         var deltaTime = time - (this.time || time);
         this.time = time;

         this.step(deltaTime);
         this.draw(deltaTime);
      }
   }
}
```

This would be acceptable in a desktop environment where you have control
of the event pump. However, in a single threaded environment like the
browser where you have to share your execution thread with the event
pump itself it gets a little bit trickier. If you block on this thread
your game will seem to have frozen, and most browsers will after a
certain period report your process as not responding, prompting the user
to kill it.

## Going Asynchronous
Thus we will have to adapt this to work with an asynchronous
approach, [setInterval] is the solution I come across most often,
it allows you to invoke a callback periodically at the given
interval. However, [setInterval] is a bad candidate because it has
the potential for the callback to be scheduled for execution before
the previous callback has completed introducing a potential [race
condition](http://en.wikipedia.org/wiki/Race_condition), the browser
will drop the scheduled invocation.

We need a way to asynchronously schedule invocation of a callback
at will, [setTimeout] allows us to do exactly that, it invokes the
given callback _once_ after the given interval, if we recursively call
setTimeout within the callback we are effectivly creating a loop that
does one iteration at a time.

```javascript
class Game {
   constructor() {
      // Holds a timestamp indicating
      // when the last tick occurred.
      // Initially set to null,
      // indicating that no tick has taken place.
      this.time = null;

      // Set a timeout to invoke this.tick as
      // soon as the browser will allow it.
      setTimeout(this.tick.bind(this), 0);
   }

   step(deltaTime) {
      // ...
   }

   draw(deltaTime) {
      // ...
   }

   tick() {
      var time = Date.now();

      // On the first tick delta time should be 0.
      var deltaTime = time - (this.time || time);
      this.time = time;

      this.step(deltaTime);
      this.draw(deltaTime);

      // Schedule this.tick to be invoked again
      // in 16 milliseconds (around 60 ticks per second).
      setTimeout(this.tick.bind(this), 16);
   }
}

```
## Requesting Animation Frames
[setTimeout] will work just fine in many cases, but here is the problem.
[setTimeout] is not precise enough, the timer resolution varies greatly
depending on the browser, anywhere from 2-20 milliseconds and really that is
just an indication, not a guarantee.

However modern browsers give us a better alternative, [requestAnimationFrame].
[requestAnimationFrame] schedules a callback invocation before the next
repaint. The number of callbacks performed is usually 60 times per second, but
will generally match the display refresh rate in most web browsers.

Also be aware that the callback rate may be reduced to a lower rate when the
process is running in the background.

```javascript
class Game {
   constructor() {
      // Holds a timestamp indicating
      // when the last tick occurred.
      // Initially set to null,
      // indicating that no tick has taken place.
      this.time = null;

      // Request an animation frame to invoke this.tick.
      requestAnimationFrame(this.tick.bind(this));
   }

   step(deltaTime) {
      // ...
   }

   draw(deltaTime) {
      // ...
   }

   tick(time) {
      // requestAnimationFrame's callback gives a very high resolution
      // timestamp (DOMHighResTimeStamp) as an argument. The timestamp
      // is accurate to a microsecond so we no longer need, nor want to
      // call Date.now as it is only accurate to the millisecond.

      // On the first tick delta time should be 0.
      var deltaTime = time - (this.time || time);
      this.time = time;

      this.step(deltaTime);
      this.draw(deltaTime);

      // Schedule this.tick to be invoked again
      // on the next animation frame.
      requestAnimationFrame(this.tick.bind(this));
   }
}
```

## Fix Your Timestep
There is still one major issue here however, and that is the naive nature of
our `tick` method.  At small intervals, everything will most likely be fine,
however as the interval between ticks increase your collisions will become
inaccurate and unstable.

We can solve this by introducing a fixed timestep.  We will keep drawing once
per frame, but update our logic at several smaller iterations per tick.

```javascript
class Game {
   constructor() {
      // Holds a timestamp indicating when the last tick occurred.
      // Initially set to null, indicating that no tick has taken place.
      this.time = null;

      // Holds the accumulative time remaining for physics steps.
      this.accumulator = 0.0;

      // Holds the size of a single timestep in milliseconds, in this
      // case we will perform around 60 steps per second.
      this.stepSize = 60 / 1;

      // Request an animation frame to invoke this.tick
      requestAnimationFrame(this.tick.bind(this));
   }

   step(deltaTime) {
      // ...
   }

   draw(deltaTime) {
      // ...
   }

   tick(time) {
      // requestAnimationFrame's callback gives a very high resolution
      // timestamp (DOMHighResTimeStamp) as an argument. The timestamp
      // is accurate to a microsecond so we no longer need, nor want to
      // call Date.now as it is only accurate to the millisecond.

      // On the first tick delta time should be 0.
      var deltaTime = time - (this.time || time);
      this.time = time;

      // Add delta time to our accumulator, iterate over the steps we
      // can do, and carry the leftovers over to the next frame.
      this.accumulator += this.deltaTime;
      while(this.accumulator >= this.stepSize) {
         this.step(this.stepSize);
         this.accumulator -= this.stepSize;
      }

      this.draw(deltaTime);

      // Request an animation frame to invoke this.tick again
      requestAnimationFrame(this.tick.bind(this));
   }
}
```

## Compatibility With Older Browsers
Arguably there is no excuse using an old browser, but not everyone
agrees so we can add at least some degree of compatibility with some
simple polyfills.

```javascript
(function () {
   window.performance = (window.performance || {});

   window.performance.now = (function () {
      return (
         window.performance.now ||
         window.performance.webkitNow ||
         window.performance.msNow ||
         window.performance.mozNow ||
         Date.now ||
         function () {
            return +new Date();
         });
   })();

   window.requestAnimationFrame = (function () {
      return (
         window.requestAnimationFrame ||
         window.webkitRequestAnimationFrame ||
         window.msRequestAnimationFrame ||
         window.mozRequestAnimationFrame ||
         function (callback) {
            return setTimeout(function () {
               var time = window.performance.now();
               callback(time);
            }, 16);
         });
   })();


   window.cancelAnimationFrame = (function () {
      return (
         window.cancelAnimationFrame ||
         window.webkitCancelAnimationFrame ||
         window.msCancelAnimationFrame ||
         window.mozCancelAnimationFrame ||
         function (id) {
            clearTimeout(id);
         });
   })();
})();
```



[setInterval]: http://docs.webplatform.org/wiki/dom/Window/setInterval "setInterval"
[setTimeout]: http://docs.webplatform.org/wiki/dom/Window/setTimeout "setTimeout"
[requestAnimationFrame]: http://docs.webplatform.org/wiki/dom/Window/requestAnimationFrame "requestAnimationFrame"