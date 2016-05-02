# Two Modes for Conflict Resolution #

MobWrite has two different modes for resolving conflicting content.  One is designed for text, the other for numeric and enum values.  The mode is chosen automatically by the client based on the content.

## Text Merging ##

Assume that Alice and Bob are collaborating on a document which currently says `B`.  Alice adds a letter to the beginning so that her text says `AB`.  Meanwhile Bob adds a letter to the end so that his text says `BC`.  When the server receives both deltas from both clients, it will merge the text changes to the best of its abilities, resulting in `ABC`.  This may or may not be what Alice and Bob intended, but it is the best solution for text merges.

A more realistic, non-minimal example would be a document which says `The cat`, Alice changes it to say `The black cat`, while Bob changes it to say `The cat meows`.  In this case the server should logically merge it to say `The black cat meows`.

## Numeric/enum Overwrites ##

Numbers ought to be merged differently than text.  Consider a document which says `2`.  Alice changes it to say `12` while Bob changes it to say `23`.  If text merging were used, the result would be `123` which would be totally incorrect in a numeric context.  In cases where the contents of an input field is just a number, MobWrite switches to a simple "last person wins" system of overwriting.

This form of overwriting is also the appropriate conflict resolution method of 'enumerated types' (enum).  Checkboxes, radio buttons and selection menus are examples of form widgets which have a set number of options and which do not respond well to creatively-merged solutions.