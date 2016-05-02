# Introduction #

This option allows for full control of MobWrite on your own server and is ideal for heavy users (Slashdot, I'm looking at you).  The principle is that your webserver (Apache, IIS, whatever) runs a tiny gateway script -- either a Python script called `q.py` or a PHP script called `q.php`.  This gateway connects to a local daemon that's listening to port 3017 ('EDIT', get it?).  The daemon keeps track of the storage and synchronization.

This two-step approach is much more efficient than doing everything in a CGI script where the data would have to be loaded and saved each time.  Use of a daemon means all the data is available at all times.

# Python vs PHP #

There is a choice of gateway: Python or PHP.  Both are functionally identical.  Which one is best depends on one's server: If mod\_python is installed, use `q.py` and the `.htaccess` handler directive.  If PHP is built-into the webserver, use `q.php`.  If neither mod\_python nor PHP is installed, and there is not the expectation of high traffic, then use `q.py` and it will degrade to CGI mode.

The situation one wants to avoid is a webserver that does not have native knowledge of a scripting language, and has to load the entire runtime environment for every web hit.  This may not be a big deal in most casual CGI work, but the gateway script gets hit a _lot_, so it is best for it to be in a native language.

# Details #

Installation is pretty easy, though it does require command-line access on your server.

  1. Download MobWrite (as daemon) from the download tab (above). Unzip it on your server.
  1. Start up the daemon with: `nohup mobwrite/daemon/mobwrite_daemon.py &`
  1. Move `mobwrite/daemon/q.py` or `q.php` to somewhere that's web-executable.
  1. Edit `mobwrite/mobwrite_core.js`, `mobwrite/tests/q.html` and `mobwrite/tests/server.html` to point at the location of the 'q' gateway.
  1. Use [JavaScript Compressor](http://dean.edwards.name/packer/) to compress `diff_match_patch_uncompressed.js`, `mobwrite_core.js`, and `mobwrite_form.js` into a single file called `compressed_form.js`.

MobWrite should now be ready.  Try the unit tests (`mobwrite/tests/index.html`) and the demos (`mobwrite/demos/index.html`) in a browser.