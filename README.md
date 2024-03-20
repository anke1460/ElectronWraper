# ElectronWraper

[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://anke1460.github.io/ElectronWraper.jl/stable/)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://anke1460.github.io/ElectronWraper.jl/dev/)
[![Build Status](https://github.com/anke1460/ElectronWraper.jl/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/anke1460/ElectronWraper.jl/actions/workflows/CI.yml?query=branch%3Amain)
[![Build Status](https://app.travis-ci.com/anke1460/ElectronWraper.jl.svg?branch=main)](https://app.travis-ci.com/anke1460/ElectronWraper.jl)
[![Build Status](https://ci.appveyor.com/api/projects/status/github/anke1460/ElectronWraper.jl?svg=true)](https://ci.appveyor.com/project/anke1460/ElectronWraper-jl)
[![Build Status](https://api.cirrus-ci.com/github/anke1460/ElectronWraper.jl.svg)](https://cirrus-ci.com/github/anke1460/ElectronWraper.jl)
[![Coverage](https://codecov.io/gh/anke1460/ElectronWraper.jl/branch/main/graph/badge.svg)](https://codecov.io/gh/anke1460/ElectronWraper.jl)
[![Coverage](https://coveralls.io/repos/github/anke1460/ElectronWraper.jl/badge.svg?branch=main)](https://coveralls.io/github/anke1460/ElectronWraper.jl?branch=main)


## Installation
FORK Electron.jl and update to electron V29.1.4
You can install the package with:

````julia
Pkg.add("ElectronWraper")
````

## Getting started

[Electron.jl](https://github.com/davidanthoff/Electron.jl) introduces two fundamental types: ``Application`` represents a running electron application, ``Window`` is a visible UI window. A julia process can have arbitrarily many applications running at the same time, each represented by its own ``Application`` instance. If you don't want to deal with ``Application``s you can also just ignore them, in that case [Electron.jl](https://github.com/davidanthoff/Electron.jl) will create a default application for you automatically.

To create a new application, simply call the corresponding constructor:

````julia
using ElectronWraper

app = Application()
````

This will start a new Electron process that is ready to open windows or run JavaScript code.

To create a new window in an existing application, use the ``Window`` constructor:

````julia
using ElectronWraper, URIs

app = Application()

win = Window(app, URI("file://main.html"))
````

Note that you need to pass a URI that points to an HTML file to the ``Window`` constructor. This HTML file will be displayed in the new window.

You can update pre-existing ``Window`` using function ``load``:

````julia
load(win, URI("http://julialang.org"))
load(win, """
<img src="https://raw.githubusercontent.com/JuliaGraphics/julia-logo-graphics/master/images/julia-logo-325-by-225.png">
""")
````

You can also call the ``Window`` constructor without passing an ``Application``, in that case [Electron.jl](https://github.com/davidanthoff/Electron.jl) creates a default application for you:

````julia
using ElectronWraper, URIs

win = Window(URI("file://main.html"))
````

You can run JavaScript code both in the main or the render thread of a specific window. To run some JavaScript in the main thread, call the ``run`` function and pass an ``Application`` instance as the first argument:

````julia
using ElectronWraper

app = Application()

result = run(app, "Math.log(10)")
````

The second argument of the ``run`` function is JavaScript code that will simply be executed as is in Electron.

You can also run JavaScript in the render thread of any open window by passing the corresponding ``Window`` instance as the first argument to ``run``:

````julia
using ElectronWraper, URIs

win = Window(URI("file://main.html"))

result = run(win, "Math.log(10)")
````

You can send messages from a render thread back to julia by calling the javascript function ``sendMessageToJulia``. On the julia side, every window has a ``Channel`` for these messages. You can access the channel for a given window with the ``msgchannel`` function, and then use the standard julia API to take messages out of this channel:

````julia
using ElectronWraper

win = Window()

result = run(win, "sendMessageToJulia('foo')")

ch = msgchannel(win)

msg = take!(ch)

println(msg)
````
