# JustEatTakeaway Ideas

## Team/Project

- daily stability meeting, why it's needed, do we have any hooks that check for anomaly?
- ways of working:
    - tools (communication), confluence,
    - definition of ready, definition of done
- JIRA no preview/code available when typing a comment
- change log for each release? - it is in confluence

## Android

### To Propose on next Sharing Session

## ideas
- optimize the startup processes by using Handlers
- linking JIRA ticket in github
- moving away from DataBinding and using ViewBinding
- SavedStateHandle injections https://insert-koin.io/docs/reference/koin-android/viewmodel/#savedstatehandle-injection-330
- why we need firebase, snagbug && kibana. Can we remove any of this, why one is better than another?
- storing amounts as float's
- build time on CI + cost?
- adding strict mode on debug
- Explicit API mode

### github actions
- remove tagging for every pipeline run, instead tag only release builds.
- https://github.blog/2023-07-19-metrics-for-issues-pull-requests-and-discussions/
- https://justeattakeaway.atlassian.net/wiki/spaces/techcomms/blog/2023/03/29/6309511581/Support+For+Gradle+Projects+In+RunSonarQubeScan+Github+Action

## proposed already
- @AllowFlaky annotation, should we use it at all - proposed ticket created
- assigning reviewers instead of "FFA".
    - proposed: mixed feeling across developers - Anfan & Anna were happy about it, rest mixed feelings.

- project structure guides & rules (architecture, naming convention, tooling) - proposed & agreed
    - we can check if lint rules are going to help us
    - we are going to keep it in the repository
- squash commits when merging to main for simpler commit history - flexibility - do it if you want proposed
- auto-deleting branches after merging - proposed and done
- review our stale branches on github - proposed and agreed.
- remove/comment flaky tests - Anfan will take care of it
- Issue with multiple activities because of `EnsureForegroundService` - resolved and committed the change.
- fixing the failing tests/jobs.

# TODOs

- check the flows from Des: "You can find everything from the link here in this screen, whilst the subsequent screens
  demonstrate everything."
  https://docs.google.com/presentation/d/1x3qY6LIsr13hiLbWH63MCDpAPnxlriD8d9NpdbeN0g0/edit#slide=id.g1b6296ba63d_1_0