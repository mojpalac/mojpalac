# Android Programming Big Nerd Ranch

you can create default style for buttons in theme

Retrofit - `getX(@Url url: String)` - allows to override base url for particular call (similarly as passing whole path
with new base url in annotation)

for images the response type has to be _ResponseBody_ and decode to `BitMap`

`@WorkerThread` allows to make sure Lint will catch and show an error when calling something on main thread

`LifecycleObserver` can be added as implementation to any object that should be aware of lifecycle. Then you can annotate fun 
e.g. `@OnLifecycleEvent(On_CREATE)` and use instance of lifecycle like: 
`this@fragment.lifyclcle.addObserver(myObserver)` make sure to call `this@fragment.lifyclcle.removeObserver()` 