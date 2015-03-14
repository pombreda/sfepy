### Installation ###

...based on discussion with Gaël Varoquaux (in 28 March 2008, check https://svn.enthought.com/enthought/wiki/MayaVi for possibly updated info)

  1. grab http://code.enthought.com/downloads/source/ets2.7/
    * install traits, then traits-ui, then mayavi
      * edit install.sh script to change installation prefix
      * run build.sh and install.sh scripts
  1. use recent wxpython (on gentoo: wxpython-2.8.7.1)
  1. use recent ipython (on gentoo: ipython-0.8.2)

then

```
$ ipython -wthread -pylab 
Python 2.4.4 (#1, Mar  6 2008, 11:24:19)
Type "copyright", "credits" or "license" for more information.

IPython 0.8.2 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object'. ?object also works, ?? prints more.

In [1]: import enthought.mayavi.mlab as M

In [2]: M.test_molecule()
Out[2]:
(<enthought.mayavi.modules.glyph.Glyph object at 0x9dd0c5c>,
 <enthought.mayavi.modules.glyph.Glyph object at 0x9e23bcc>,
 <enthought.mayavi.modules.glyph.Glyph object at 0x9e2fadc>,
 <enthought.mayavi.modules.glyph.Glyph object at 0x9e3b9ec>)

In [3]: M.show_engine()
Out[3]: <enthought.traits.ui.ui.UI object at 0x9ee132c>
```

## Installation on Debian ##

```
$ apt-get install mayavi2
```

And that's it.

```
$ ipython -wthread -pylab
Python 2.4.5 (#2, Mar 12 2008, 00:15:51) 
Type "copyright", "credits" or "license" for more information.

IPython 0.8.2 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object'. ?object also works, ?? prints more.

In [1]: import enthought.mayavi.mlab as M

In [2]: M.test_molecule()

(python:4497): Gtk-CRITICAL **: gtk_widget_set_colormap: assertion `!GTK_WIDGET_REALIZED (widget)' failed
Warning: invalid value encountered in divide
Out[2]: 
(<enthought.mayavi.modules.glyph.Glyph object at 0x9a6eefc>,
 <enthought.mayavi.modules.glyph.Glyph object at 0x9ad6c8c>,
 <enthought.mayavi.modules.glyph.Glyph object at 0x9ac7b9c>,
 <enthought.mayavi.modules.glyph.Glyph object at 0x9ad3aac>)

In [3]: M.show_engine()
Out[3]: <enthought.traits.ui.ui.UI object at 0xa0ad3ec>
```