# Technical Overview #

[Differential synchronization](http://neil.fraser.name/writing/sync) - MobWrite is a symmetrical system where the client and the server run exactly the same algorithms, sharing the work in keeping everyone in sync.  Differential synchronization provides a fault-tolerant system that allows conflicts to be resolved automatically on a best-effort basis.

[Diff, Match and Patch](http://code.google.com/p/google-diff-match-patch/) - At the core of both the client and the server is an efficient library that identifies local changes then merges remote changes into the local content.  The matching algorithms are also used to restore the cursor or selection after remote changes are received.

[Communication protocol](Protocol.md) - Synchronizations occur every few seconds, with the frequency automatically increasing during periods of activity and decreasing when idle.  Data is transmitted across the web using Ajax requests, formatted to a custom protocol.  This protocol is intended to be simple and flexible enough that new clients or servers may be created which integrate cleanly with MobWrite.

[Text vs. Numbers](Numeric.md) - Two different conflict resolution strategies are used.  Conflicting text changes are merged gently, attempting to incorporate the intention of all parties.  While there exist no-win scenarios, MobWrite does the best job it can, with the understanding that an imperfect merge done in real time will be seen immediately by all affected parties.  This is distinct from numeric data and enumerated types which are simply overwritten with the last selected value.

[Cursor Preservation](http://neil.fraser.name/writing/cursor/) - When a remote update is received the local user's cursor, selection and scrollbar needs to be adjusted carefully o make the update as smooth as possible.  Two algorithms are blended together to achieve this.