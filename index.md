---
layout: page
---

The [Southeast USA Region](https://ser.cs.fit.edu/) of the ACM ICPC has been
working hard for many years on the linux environment our teams use during our
competition. Rather than keep this work all to ourselves, we want to share
it with the world!

To this end, we have published the entire set of tools and scripts we use to
build our image from scratch. We are also in the process of publishing many
notes about our image on this site, so even if you don't directly use it you
may be able to benefit from some of the solutions we've come up with.

You can find the tools repository here: [icpc-env github repository](https://github.com/icpc-environment/icpc-env)  
Notes and various discussions of problems and solutions are found here: [blog](/blog/)

## Key Features
Our contest environment has been tuned and tweaked over the years, but it
currently has the following:

* Support for 14 programming languages: Ada, C, C++, C#, Fortran, Go, Haskell, Java, Lua, Pascal, Python 2, Python 3, Ruby, Scala
* Multiple IDEs and developer tools: [Eclipse JDT](http://eclipse.org/jdt/)(with [PyDev](http://www.pydev.org/)/[CDT](https://eclipse.org/cdt/)), [MonoDevelop](http://www.monodevelop.com/), [Code::Blocks](http://www.codeblocks.org/), [gvim](http://www.vim.org/), [emacs](https://www.gnu.org/software/emacs/), [atom](https://atom.io/), [gedit](https://wiki.gnome.org/Apps/Gedit), [geany](http://www.geany.org/)
* Local language documentation for: C++ STL, Scala, Java, Python2/3, Pascal, Haskell
* Optimized for USB flash drives ([Details]({% post_url 2015-10-26-usb-optimization %}))
* Supports DOMjudge automatic login ([Details]({% post_url 2015-10-24-domjudge-autologin %}))
* Advanced firewall to restrict team access to the network ([Details]({% post_url 2015-10-27-team-firewall %}))
* Printing support
* No modification to the physical machine required
* Fat32 partition for teams to store files that allows for easy access after the contest
* Fully customizable, entirely automated process for building consistent images
* Lightweight XFCE window manager
* Supports 32 or 64 bit machines

All this fits on a good 8gb flash drive you can buy for [less than $5](http://www.newegg.com/Product/Product.aspx?Item=N82E16820239764).


## Custom Images
If you don't have the know how or time to put together an image for your contest
yourself, we may be able to help you out with a custom image(free of charge).
Contact Keith Johnson(kj at ubergeek42.com) for information and details.
