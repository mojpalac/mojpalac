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
- @AllowFlaky annotation, should we use it at all
- optimize the startup processes by using Handlers
- Maybe review how are we doing it(as a part of github actions to improve performance)?
- https://justeattakeaway.atlassian.net/wiki/spaces/techcomms/blog/2023/03/29/6309511581/Support+For+Gradle+Projects+In+RunSonarQubeScan+Github+Action
- linking JIRA ticket in github
- Explicit API mode
- moving away from DataBinding and using ViewBinding
- why we need firebase, snagbug && kibana. Can we remove any of this, why one is better than another?
- storing amounts as float's
- build time on CI + cost?
- adding strict mode on debug 
- SavedStateHandle injections https://insert-koin.io/docs/reference/koin-android/viewmodel/#savedstatehandle-injection-330


## proposed already
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