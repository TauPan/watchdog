Watchdog
========
Python API and shell utilities to monitor file system events.

Example API Usage:
------------------

A simple program that uses watchdog to monitor directories specified
as command-line arguments and logs events generated.

	import sys
	import time
	from watchdog import Observer
	from watchdog.events import FileSystemEventHandler
	import logging

	logging.basicConfig(level=logging.DEBUG)

	class MyEventHandler(FileSystemEventHandler):
	    def catch_all_handler(self, event):
	        logging.debug(event)

	    def on_moved(self, event):
	        self.catch_all_handler(event)

	    def on_created(self, event):
	        self.catch_all_handler(event)

	    def on_deleted(self, event):
	        self.catch_all_handler(event)

	    def on_modified(self, event):
	        self.catch_all_handler(event)


	event_handler = MyEventHandler()
	observer = Observer()
	observer.schedule('a-unique-name', event_handler, sys.argv[1:], recursive=True)
	observer.start()
	try:
	    while True:
	        time.sleep(1)
	except KeyboardInterrupt:
	    observer.unschedule('a-unique-name')
	    observer.stop()
	observer.join()


Shell Utilities:
----------------
Watchdog comes with a utility script called `watchmedo`.
Please type `watchmedo --help` at the shell prompt to
know more about this tool.

Here is how you can log the current directory recursively
for events related only to `*.py` and `*.txt` files while
ignoring all directory events:

    watchmedo log --patterns="*.py;*.txt" --ignore-directories --recursive .

If you'd like to execute shell commands in response to
events you can use the `shell-command` subcommand like this:

    watchmedo shell-command --patterns="*.py;*.txt" --recursive --command='echo "${watch_src_path}"' .

Please see the help information for these commands by typing:

    watchmedo [command] --help

About `watchmedo` Tricks:
~~~~~~~~~~~~~~~~~~~~~~~~~
Watchmedo can read "tricks.yaml" files and execute tricks
within them in response to file system events. Tricks are
basically FileSystemEventHandlers that plugin authors can
write. Trick classes are augmented with a few additional
features that regular event handlers don't need.

An example `tricks.yaml` file:

    python-path: [app, foobar/bin, whatever/another/directory]
    watches:
      - app/static
    tricks:
    - watchdog.tricks.LoggerTrick:
        kwargs:
          patterns: ["*.py"]
          ignore_patterns: ["*.pyc", "*.pyo"]
          ignore_directories: True
    - watchmedo_webtricks.GoogleClosureTrick:
        kwargs:
          patterns: ['app/static/js/*.js']
          hash_names: true
          mappings_module: app/javascript_mappings.py
          suffix: .min.js
          compilation_level: advanced            # simple|advanced
          source_directory: app/static/js/
          destination_directory: app/public/js/
          files:
            index-page:
            - app/static/js/vendor/jquery.js
            - app/static/js/base.js
            - app/static/js/index-page.js
            about-page:
            - app/static/js/vendor/jquery.js
            - app/static/js/base.js
            - app/static/js/about-page.js

Each trick class is initialized with the given kwargs
keyword arguments and events are fed to an instance of
the class as they arrive.

Tricks will be included in the 0.4.0 release. I need community
input about them. Please file enhancement requests at the
[issue tracker](http://github.com/gorakhargosh/watchdog/issues').


Installation:
-------------

Installing from PyPI using pip:

    pip install watchdog

Installing from PyPI using easy_install:

    easy_install watchdog

Installing from source:

    python setup.py install


Supported Platforms:
--------------------

* Linux 2.6 (inotify)
* Mac OS X (FSEevnts, kqueue)
* FreeBSD/BSD (kqueue)
* Windows (ReadDirectoryChangesW with I/O completion ports; ReadDirectoryChangesW worker threads)
* OS-independent (polling the disk for directory snapshots and comparing them periodically; slow and not recommended)


Dependencies:
-------------
1. Python 2.5 or above.
2. [pywin32](http://sourceforge.net/projects/pywin32/) (only on Windows)
3. [pyinotify](http://github.com/seb-m/pyinotify) (only on Linux)
4. [XCode](http://developer.apple.com/technologies/tools/xcode.html) or gcc (only on Mac OS X)
5. [PyYAML](http://www.pyyaml.org/)
6. [argh](http://pypi.python.org/pypi/argh)
7. [select_backport](http://pypi.python.org/pypi/select_backport/) (select.kqueue replacement for Python2.5/2.6 on BSD/Mac OS X)


Licensing:
----------
Watchdog is licensed under the terms of the
[MIT License](http://www.opensource.org/licenses/mit-license.html)

Copyright (C) 2010 Gora Khargosh &lt;[gora.khargosh@gmail.com](mailto:gora.khargosh@gmail.com)&gt; and the Watchdog authors.

Project source code at [Github](http://github.com/gorakhargosh/watchdog)

Please report bugs at the [Github issue tracker](http://github.com/gorakhargosh/watchdog/issues).
