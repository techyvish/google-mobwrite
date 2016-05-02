# Introduction #

Synchronization and automatic conflict resolution are hard problems.  Most web applications are consciously or unconsciously designed to separate users from each other, thus minimizing the number of collisions.  MobWrite allows web applications to seamlessly connect users to each other.  Just build your web application for a single user, add two lines of JavaScript and some appropriate IDs to the form elements, and you're done.

# Client-side #

Add the following JavaScript:
```
<SCRIPT SRC="http://mobwrite3.appspot.com/static/compressed_form.js"></SCRIPT>`
<SCRIPT>
  mobwrite.syncGateway = 'http://mobwrite3.appspot.com/scripts/q.py';
</SCRIPT>
<BODY ONLOAD="mobwrite.share('formid');">
```

[Detailed instructions](WebClient.md).

Note that MobWrite has _no authentication_.  That's the job of the host application.  MobWrite is simply a sharing pipe that the host application uses to connect forms with each other.  To keep unwanted users out and to prevent collisions, it is recommended to use form element IDs that are eight-character random strings.  If all users of a form are to be globally connected (as in the demos), then one can just hard-code random IDs.  If the users are to be broken into groups, with data only syncing between members of the group, then the IDs must be generated so that members of the group all have the same IDs.

# Server-side #

There are three ways to setup a server.

## 1: MobWrite service ##

MobWrite is running on Google App Engine as `mobwrite3.appspot.com`.  If your loads are relatively light, you are welcome to use this service.  In this case you don't need to do anything.

## 2: App Engine ##

The next heavier step is to create your own account on Google App Engine and upload your own copy of MobWrite.  This also allows you to customize MobWrite if needed. [Detailed instructions](AppEngine.md).

## 3: Python Daemon ##

For heavy loads (did you just max-out your Google App Engine account?) you can download the MobWrite code and set it up as a Python daemon running on your own web server.  [Detailed instructions](Daemon.md).