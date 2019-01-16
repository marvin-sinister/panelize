# KiCAD panelize

This was originally written by [Martin Furter](mailto:mf@borg.ch) as part of his [Tin's Kicad Tools](https://projects.borg.ch/electronics/kicad/panelize.html). I took the tool and invested minimum possible effort to make ti compatible with KiCAD 5.

panelize.py is a tool to panelize kicad PCBs. This version supports kicad format 4 PCB files, those with the ending ".kicad_pcb". It can copy and optionally rotate and flip parts of PCB files. It can also be used to merge multiple PCB files. It works with KiCAD 5, and should work with KiCAD 4.

## Index

* [Usage](#usage)
* [Configfile](#configfile)
  * [Layer names](layer-names)
  * [Commands](#commands)
    * [new](#new)
    * [load](#load)
    * [save](#save)
    * [compat](#compat)
    * [create-template](#create-template)
    * [source-area](#source-area)
    * [layer commands](#layer commands)
    * [titleblock commands](#titleblock-commands)
    * [clone-nets](#clone-nets)
    * [swap-internal-layers](#swap-internal-layers)
    * [Copy Commands](#copy-commands)
    * [draw-line](#draw-line)
    * [draw-text](#draw-text)
    * [set-layer](#set-layer)
    * [set-line-thickness](#set-line-thickness)
    * [set-text-font](#set-text-font)
  * [Example Configfile](#example-configfile)
    * [Simple Example](#simple-example)
    * [A Bigger Example](#a-bigger-example)

## Usage

`# panelize.py configfile`

The configfile contains all commands for the panelisation.

## Configfile

The config file is a simple text file. It contains one command per line. Empty lines and lines starting with # are ignored. Commands start with a keyword followed by the paramters. Strings can be enclosed in single or double quotes. All measurements are in millimeters.

### Layer names:

    F.Cu
    In1.Cu In2.Cu ... In30.Cu
    B.Cu
    B.Adhes
    F.Adhes
    B.Paste
    F.Paste
    B.SilkS
    F.SilkS
    B.Mask
    F.Mask
    Dwgs.User
    Cmts.User
    Eco1.User
    Eco2.User
    Edge.Cuts
    Margin
    B.CrtYd
    F.CrtYd
    B.Fab
    F.Fab

### Commands

#### new

    new

Creates a new empty destination PCB.

##### Example:

    new

#### load

    load <filename>

Loads a kicad PCB file and uses it as source for copies. If no destination PCB exists the loaded PCB is also used as destination.

##### Example:

    load example.kicad_pcb

#### save

    save <filename>

Saves the destination PCB to the specified file.

##### Example:

    save example.kicad_pcb

#### compat

    compat 4.0.5

Save the PCB file in a version compatible to the specified version of kicad. Currently only the versions "4.0.5" and "latest" are supported.

#### create-template

    create-template

Creates a template from the source PCB and sets it as destination PCB. The template is a copy of the source containing all the configuration parameters, but no modules, traces, etc. are copied.

##### Example:

    create-template

#### source-area

    source-area <X1> <Y1> <X2> <Y2>

Sets the rectangular area in the source PCB for copies.

##### Example:

    source-area 50 50 74 100

#### layer commands

    exclude-layer <layer-name>

    include-layer <layer-name>

    include-layer all

Includes or excludes the specified layer from the copy. Multiple layers can be included or excluded by executing multiple include or exclude commands. Switching from include to exclude or vice versa clears the previous list.

##### Example:

    exclude-layer Edge.Cuts

#### titleblock commands

    set-title <text>

    set-date <YYYY-MM-DD>

    set-rev <text>

    set-company <text>

    set-comment-1 <text>

    set-comment-2 <text>

    set-comment-3 <text>

    set-comment-4 <text>

these commands set the texts in the title block.

##### Example:

    set-title 'Panelized PCB'

#### clone-nets

    clone-nets <bool>

When set to true the nets of the copies will be duplicated. Their names will have a prefix "C" followed by the number of the copy command and the original name. When nets are not cloned the resulting PCB will show a ratsnest for the missing connections between the PCBs.

##### Example:

    clone-nets false

#### swap-internal-layers

    swap-internal-layers <bool>

When set to true internal layers will be swapped too when fliping a PCB. That means internal layer 1 becomes layer N, 2 becomes N-1, etc.

##### Example:

    swap-internal-layers true

#### Copy Commands

    copy <X> <Y>
    rotate-right <X> <Y>
    rotate-180 <X> <Y>
    rotate-left <X> <Y>
    flip-copy <X> <Y>
    flip-rotate-right <X> <Y>
    flip-rotate-180 <X> <Y>
    flip-rotate-left <X> <Y>

Copies the selected area to the given destination and optionally rotates it or flips it to the other PCB side. The coordinate is the upper left corner of the destination.

##### Example:

    copy 76 50

#### draw-line

    draw-line <x1> <y1> <x2> <y2> [layer-name [thickness]]

Draw a line. If layer or thickness are not specified a default is used.

##### Example:

    draw-line 75 50 75 100
    draw-line 75 50 75 100 Eco1.User 0.5

#### draw-text

    draw-text text <x> <y> [angle [layer-name [height [width [thickness]]]]]

Draw some text. For the arguments which are not specified a default is used.

##### Example:

    draw-text horizontal 75 105
    draw-text vertical 90 105 90 Eco1.User 3 2 0.25

#### set-layer

    set-layer <layer-name>

Sets the layer name which is used as default for the draw commands.

##### Example:

    set-layer Eco1.User

#### set-line-thickness

    set-line-thickness <thickness>

Sets the thickness which is used as default for the draw-line command.

##### Example:

    set-line-thickness 0.3

#### set-text-font

    set-text-font

Sets the font (charater height and width, line thickness) which is used as default for the draw-text command.

##### Example:

    set-text-font 1.5 1.2 0.15

### Example Configfile

#### Simple Example
```
load example.kicad_pcb
source-area 50 50 74 100
copy 76 50
save panelized.kicad_pcb
```

#### A Bigger Example

```
# options
clone-nets true
exclude-layer Edge.Cuts

# rpi-quad-serial-panel-border: 54.5/54.5..119.5/110.5
load rpi-quad-serial-panel-border.kicad_pcb

# rpi-quad-serial: 54.5/54.5..119.5/110.5
load rpi-quad-serial.kicad_pcb
source-area 54.4 54.4 119.6 110.6
copy 54.4 54.4
include-layer Edge.Cuts
source-area 66 87 102 109
copy 66 87
exclude-layer Edge.Cuts

# ttl-2-rs232-isol.kicad_pcb: 89/88..111/150
load ttl-2-rs232-isol.kicad_pcb
source-area 89 88 111 150
copy 121 54.5

# ttl-2-rs485-isol.kicad_pcb: 
load ttl-2-rs485-isol.kicad_pcb
source-area 89 88 111 150
rotate-right 54.5 112

# save...
save rpi-quad-serial-panel.kicad_pcb
```

