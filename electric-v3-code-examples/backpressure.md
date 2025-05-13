# Backpressure and Concurrency <span id="title-extra"><span>

<div id="nav"></div>

"UI is a concurrency problem" — Leo Noel

This is just the Two Clocks demo with slight modifications, there is more to learn here.

!ns[electric-tutorial.backpressure/Backpressure]()

What's happening
* The timer `e/system-time-secs` is a float and updates at the browser animation rate, let's say 120hz.
* Clocks are printed to both the DOM (at 120hz) and also the browser console (at 1hz)
* The clocks pause when the browser page is not visible (i.e. you switch tabs), confirm it **[todo electric v3]**

Key ideas
* **work-skipping**: expressions are only recomputed if an argument actually changes, otherwise the previous value is reused. Here, the truncated clock timer does not needlessly spam the prn; downstream nodes are only recomputed on the transition.
* **signals**: Electric reactive functions model *signals*, not *streams*. Streams are like mouse clicks or database transactions – you can't skip one. Signals are like mouse coordinates, audio signals or game animations – you mostly only care about latest, nobody wants an animation frame from 2 seconds ago.
* **the reactive clocks are lazy**: the clocks do not tick until the previous tick was *sampled* (i.e. consumed). That means if the rendering process falls behind the requestAnimationFrame tick rate, it will simply skip frames like a video game.
* **lazy sampling**: All electric expressions are lazily sampled, not just the clocks. Electric fns don't compute anything at all until sampled by the Electric entrypoint.

Low-level concurrency control

* If/when you do want to play at the functional reactive layer underneath Electric, we have the right trapdoors for that.
* `e/System-time-ms` is implemented using Missionary. It's too advanced to show here, but inside this function we use Missionary to implement an efficient backpressured clock that only ticks when someone is looking. Low level concurrency control at every point in the computation is a first class feature of Electric.
* With respect to types: if everything's reactive, nothing's reactive, all the FRP types cancel out. With Electric you will never make a type error related to async vs sync types; forget about them.

Clock details
* `e/system-time-secs` is implemented as a missionary flow.
* on the client, the underlying clock is implemented with `requestAnimationFrame` such that it only schedules work once the current tick has been sampled.
* That means, if you switch to another tab, the browser will stop scheduling animation frames and the clock will pause.
* The server clock is implemented similarly, it will tick as fast as possible but only when sampled. If nobody is consuming the clock — i.e. no websockets are connected — the clock will not tick at all!

Work-skipping
* We use `int` to truncate the precision. The truncation is performed at 120hz, as we want to switch *precisely* on the rising edge of the transition.
* Since `int` is recomputing at 120hz, that means anything immediately downstream will be checked 120 times per second
  * so both `(prn "s" (int s))` and `(- s c)` are both checked at 120hz
* However, expressions only run if at least one argument has changed
* So, at this point the memoization kicks in and we will skip the work, this is called "work-skipping"
* `(prn "s" (int s))` prints at 1hz not 120hz

Lazy sampling
* Q: Truncating at 120hz is inefficient right? Why not use `(js/setTimeout 1000)` to make a slower clock?
* A: Actually, the fast clock is efficient due to Electric's backpressure.
* If the rendering process can't keep up with the requestAnimationFrame tick rate, it will simply skip frames, and this will actually slow down the clock because the clock itself is lazy too.
* This is a form of backpressure. See [Streams vs Signals](https://www.dustingetz.com/#/page/signals%20vs%20streams%2C%20in%20terms%20of%20backpressure%20%282023%29)
* Note: Computers can do 3D transforms in realtime, running this little clock at the browser animation rate is negligible.
* If your computer is powered on and plugged in, you generally want to render at the highest framerate the device is capable of. This is Electric's default behavior out of the box.

Backpressure
* What if you're on an old mobile device that can't keep up with the server clock streaming?
  * In this case, incoming messages from server will saturate the websocket buffer,
  * then the browser will propagate backpressure at the TCP layer,
  * then the server will decrease the *sampling rate*.
* Same behavior in the opposite direction (e.g if the client generates lots of events and the server is overloaded).
* See: [Signals vs Streams, in terms of backpressure](https://www.dustingetz.com/#/page/signals%20vs%20streams%2C%20in%20terms%20of%20backpressure%20%282023%29)
* Electric is a Clojure to [Missionary](https://github.com/leonoel/missionary) compiler; it compiles lifts each Clojure form into a Missionary continuous flow.
* Thereby automatically backpressuring every point in the Electric DAG.
* If you want to customize the backpressure locally at any point — maybe you want to run a chat view with realtime network but run a slow query less aggressively — you can drop down into missionary's rich suite of concurrency combinators.

Beginners can get away with pretending your program is a Clojure program for a bit, but you'll eventually need to reason about Electric programs as DAGs†:

* Each expression, e.g. `(- s c)`, is a node in the DAG.
* Expression arguments (e.g. `s`) are edges in the DAG.
* Electric `let` binds names (e.g. `s`) to evaluation results in the DAG, and memoizes the result for sharing and reuse in multiple downstream consumers (i.e. expressions).
* So for example, node `(- s c)` has:
  * three incoming edges `- s c`, and
  * one outgoing edge—the result—consumed by `(dom/text "skew: " _)`
  * So, if `s` changes, `(- s c)` is recomputed using memoized `c`, and then the `dom/text` reruns (a point write).