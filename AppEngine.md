# Introduction #

Google offers a web application hosting service.  There are no costs, no ads, no spam.  The only catch is that there are quotas for how much traffic, storage and CPU time one can consume.  If your application isn't Slashdot, you shouldn't have a problem.

# Details #

  1. Register an account with [Google App Engine](http://appengine.google.com/) and download the SDK.
  1. Download MobWrite (for App Engine) from the download tab (above).  Unzip it on your computer.
  1. Edit `app.yaml` to change the application name.
  1. Use the SDK to upload your copy of MobWrite: 'appcfg.py update /path/to/mobwrite/'
  1. If your web client is not hosted on this App Engine instance, then edit both the JavaScript include and gateway property in your client to point at your new App Engine instance.  [Detailed instructions](WebClient.md).

MobWrite should now be synchronizing.  If not, try the unit tests (`http://<appname>.appspot.com/static/tests/index.html`) and the demos (`http://<appname>.appspot.com/static/demos/index.html`).