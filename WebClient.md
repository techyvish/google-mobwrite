# Introduction #

Setting up a web client is trivial, just add the following to the page:
```
<SCRIPT SRC="http://mobwrite3.appspot.com/static/compressed_form.js"></SCRIPT>`
<SCRIPT>
  mobwrite.syncGateway = 'http://mobwrite3.appspot.com/scripts/q.py';
</SCRIPT>
<BODY ONLOAD="mobwrite.share('formid');">
```

MobWrite has a number of client-side functions and parameters which may be used for more advanced cases.

# Functions #

```
mobwrite.share(element1, element2, ...);
mobwrite.share('id1', 'id2', ...);
```
The share function takes form elements (text areas, check boxen, etc) or their IDs.  This function can also take forms or the form ID, in which case all contained form elements will be shared (including hidden fields).  The only form element which cannot be shared is a file-input field.  All form elements shared must have an ID.  Sharing a field multiple times does no harm.

Clients do not need to share the same list of elements as each other.  It is perfectly acceptable for one client to share fields A, B and C, while another client shares B, C and D.  In this case only B and C will be linked between clients.

```
mobwrite.unshare(element1, element2, ...);
mobwrite.unshare('id1', 'id2', ...);
```
Stops sharing the requested element(s).  Unlike share(), unshare() does not currently accept forms as input.  If you want to stop sharing everything but leave the fields in place on the server, simply set `mobwrite.shared = {}`.  See also the `nullifyAll` property below.

After a remote change arrives from the server, the element's `onchange` event handler will be fired.  This allows a page to react to changes.  See the checkbox on the [form demo](http://mobwrite3.appspot.com/static/demos/form.html) as an example.

# Properties #

```
mobwrite.syncGateway = '/scripts/q.py';
```
The gateway is the URL where the synchronizations should be sent.  If this is a relative URL then quiet Ajax connections will be used, otherwise JSONP will be used.  JSONP will defeat the same-origin policy but has several disadvantages, such as causing the status bar to flicker and transmitting all outbound data on the URL.

```
mobwrite.minSyncInterval = 1000;
```
Shortest interval (in milliseconds) between connections.  If there is activity on the client or server, then the frequency between synchronizations will decrease until minSyncInterval is reached.

```
mobwrite.maxSyncInterval = 10000;
```
Longest interval (in milliseconds) between connections.  If there is no activity on either client or server, then the frequency between synchronizations will increase until maxSyncInterval is reached.

```
mobwrite.syncInterval = 2000;
```
Initial interval (in milliseconds) for connections.  This value is modified later as traffic rates are established.

```
mobwrite.timeoutInterval = 30000;
```
Time to wait (in milliseconds) for an Ajax or JSONP connection before giving up and retrying.

```
mobwrite.idPrefix = '';
```
Optional prefix to automatically add to all IDs.  If set to "group7" before sharing a field named "title", then the server will store a field named "group7title".  Being able to add a common prefix to all field names may be useful in establishing sharing groups which are isolated from each other.  This is merely a convenience property, the same effect can be obtained by merely naming the field's ID to be "group7title".

```
mobwrite.nullifyAll = false;
```
When set to true, the client's next sync will issue nullification requests to all shared files on the server.  This property may be used by a cancel button so that the next time someone visits this form the existing text is not repopulated.  Note that MobWrite adds a synchronization request on the page's unload event, so setting nullifyAll to true and leaving the page should be enough to purge the data from the server.  If collaborators are currently looking at the form, their clients will quietly repopulate the fields.

```
mobwrite.debug = true;
```
In Firefox (with Firebug), Safari, Chrome and other browsers with a console, switching on debug mode will spew lots of status messages.  Tip: you might want to set this debug flag one way or the other since the default seems to be inconsistent between releases.