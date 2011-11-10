## MyQt4
### PyQt4 package builder for Maya 2012

With the many tutorials and websites documenting various ways to install PyQt4 for Maya, and the complications it poses
for the less tech-savvy, it only makes sense that the process should be wrapped into a build script and a package installer.
I have seen precompiled packages for windows, but not yet for OSX or Linux.

### Building (osx)

This will produce a .pkg that can be distributed and installed, for Maya 2012. It assumes the standard install location of
Maya 2012-x64

        git clone git://github.com/justinfx/MyQt4.git
        cd MyQt4/osx
        make

### Hosted builds

Latest OSX pkg build can be downloaded here:<br>
http://dl.dropbox.com/u/34613220/MyQt4/MyQt4.8.6-maya2012-x64-osx.pkg

### Notes about installer

* Installes a repaired pyuic4 as /usr/local/bin/myqt4
* Copies pyrcc4 to /usr/local/bin/pyrcc4
        

        