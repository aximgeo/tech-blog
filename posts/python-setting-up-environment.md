# Python Development with ArcGIS, Part 1:  Setting up your Environment
If you work with ArcGIS regularly, chances are you've been exposed to Python--it's the scripting language of choice for most ESRI environments.  But even with a clean ArcGIS install, there are a few things you can do to make developing in Python a more pleasant experience:

* Adding Python to your PATH
* Installing a package manager
* Upgrading your editor

## Adding Python to your PATH
By default, Python is not added to your system `PATH` when you install ArcGIS; since we'll often be running things from a command prompt, we need to make sure that Python is available.  A quick check is to open your prompt ("cmd" in the windows search bar), and simply enter `python`.  You should see something like this:

```
ifirkin@D7510-1613 c:\develop
> python
Python 2.7.8 (default, Jun 30 2014, 16:03:49) [MSC v.1500 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

## Installing pip
[pip](https://pip.pypa.io/en/stable/) is python's default package manager, similar to npm, nuget, or chocolatey.  If you're running ArcGIS 10.4 or above, it's already installed on your system.  For previous versions, you'll need to download the [installation script](https://bootstrap.pypa.io/get-pip.py) and run: 

```
python get-pip.py
```

## Upgrading your Editor
By default, ArcGIS ships with PythonWin as an editor; while it's functional, there are many better options today.  If you want a full-featured, all-in-one IDE, [PyCharm](https://www.jetbrains.com/pycharm/download/#section=windows) may be a good fit; if you're already using [Visual Studio Code](https://code.visualstudio.com/) there are good Python [extensions](https://code.visualstudio.com/docs/languages/python) available.
