# Bugsnag

## tips

- Make use of the Timeline view as our default maybe. This has all the same information as Inbox but allows us to more
  easily see when certain issues are arising, or notice visual patterns like we do in Grafana
- Make use of isLaunching and other flags which give us more detail about app state during a crash. See if we spot any
  trends, and if we do then bugsnag has hooks for making use of these at runtime
- Make more use of custom filters as these are exposed in the "pivot table" (device / app state info in bottom left of
  issue view) which gives us more information about which devices are having these issues at a glance
- We can make use of the "Features" tab and link this to JETFM config via OP so that we can detect bugs in particular
  experiments more easily
- We can all make use of the "New error view" button to create custom widgets for ourselves to track issues arising in
  certain app areas / situations (e.g. when navigating to a given destination) and these work on a per-user basis so
  we're not impacting eachother so can use this liberally
- We could make use of assignement rules so that issues that arise in a particular package or class can notify a
  particular engineer, e.g. if one of us makes a refactor, we could assign that code area to that engineer and then
  they'd be notified more quickly of any introduced issues