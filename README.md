Handbrake for Ruby
==================

This library provides a lightweight literate ruby wrapper for
[HandBrakeCLI][], the command-line interface for the [HandBrake][]
video transcoder.

[HandBrakeCLI]: https://trac.handbrake.fr/wiki/CLIGuide
[HandBrake]: http://handbrake.fr/

The intent of this library is to make it a bit easier to script
HandBrake. You will still need to be familiar with [HandBrake][] and
[HandBrakeCLI][] to make use of it.

Prerequisites
-------------

* [HandBrake][hb-dl] and [HandBrakeCLI][cli-dl] (tested with 0.9.5)
* Ruby and rubygems (tested with 1.8.7, but expected to work elsewhere)

Installation
------------

`handbrake-ruby` is distributed as a rubygem:

    $ gem install handbrake

Use
---

A brief sample:

    require 'handbrake'

    hb = HandBrake::CLI.new(:bin_path => '/Applications/HandBrakeCLI', :trace => false)

    project = hb.input('/Volumes/Arcturan Megafreighter/DVDs/A/VIDEO_TS')

    titles = project.scan
    titles[0].main_feature?       # => true
    titles[0].duration            # => "01:21:18"
    titles[0].seconds             # => 4878
    titles[0].chapters.size       # => 23
    titles[0].chapters[2].seconds # => 208
    titles[0].number              # => 1

    project.title(1).
      preset('Normal').
      output('/Users/rsutphin/Movies/project.m4v').

In additional detail:

### Create a HandBrake::CLI instance

    require 'handbrake'

    hb = HandBrake::CLI.new(:bin_path => handbrake_cli_path, :trace => true)

This object carries the path to the HandBrakeCLI bin and other library
configuration options:

* `:bin_path`: the path to the `HandBrakeCLI` executable. The default
  is `'HandBrakeCLI'`; i.e., by default it will be searched for on the
  normal executable path.
* `:trace`: if true, all output from `HandBrakeCLI` will be echoed to
  the project's output stream.

### Set options

You build up a command string by invoking a chain of methods starting
from a `HandBrake::CLI` instance. The methods are named following the
long form of the options for [HandBrakeCLI][]. (The one exception to
this naming scheme is for options that contain a dash; in those cases,
an underscore must be substituted for the dash.)

E.g., the HandBrakeCLI documentation has this command:

    $ HandBrakeCLI -i /Volumes/MyBook/VIDEO_TS -o /Volumes/MyBook/movie.m4v -v -P -m -E aac,ac3 -e x264
      -q 0.65 -x ref=3:mixed-refs:bframes=6:b-pyramid=1:weightb=1:analyse=all:8x8dct=1:subme=7:me=umh
      :merange=24:filter=-2,-2:trellis=1:no-fast-pskip=1:no-dct-decimate=1:direct=auto

In `handbrake-ruby`, you could build up this command like so:

    vid_opts = 'ref=3:mixed-refs:bframes=6:b-pyramid=1:weightb=1:analyse=all:8x8dct=1:subme=7:me=umh:merange=24:filter=-2,-2:trellis=1:no-fast-pskip=1:no-dct-decimate=1:direct=auto'
    HandBrake::CLI.new.input('/Volumes/MyBook/VIDEO_TS').verbose.
      loosePixelratio.markers.aencoder('aac,ac3').encoder('x264').
      quality('0.65').x264opts(vid_opts).
      output('/Volumes/MyBook/movie.m4v')

The `output` option has to go last; see the next section for more details.

### Starting execution

While most of the methods you call to build up a handbrake command can
come in any order, a few must come last and have particular return
values:

* `output`: triggers a transcode using all the options set up to this
  point.
* `scan`: triggers a title scan and returns an `Array`-like structure
  of {HandBrake::Title} objects describing the contents of the
  input. Note that, per usual ruby practices, this array will use
  zero-based indicies.
* `update`: returns true or false depending on whether the version of
  HandBrakeCLI in use is up to date.
* `preset_list`: returns a hash containing all the known presets and
  their options. The structure is `presets[category][name] => args`

### Reusing a configuration chain

At any point before invoking one of the execution methods (listed in
the previous section), you can save off the chain and continue it
along different paths.  E.g.:

    project = HandBrake::CLI.new.input('VIDEO_TS')

    # iPhone
    project.preset('iPhone & iPod Touch').output('project-phone.m4v')

    # TV
    project.preset('High Profile').output('project-tv.m4v')

To put it more technically, the chain object returned from each
intermediate configuration step returns an independent copy of the
configuration chain.

Additional resources
--------------------

License
-------

    handbrake-ruby
    Copyright (C) 2011 Rhett Sutphin.

    This library is free software; you can redistribute it and/or
    modify it under the terms of the GNU Lesser General Public
    License as published by the Free Software Foundation; either
    version 2.1 of the License, or (at your option) any later version.

    This library is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
    Lesser General Public License for more details.
