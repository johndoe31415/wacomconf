# wacomconf
Configure a Wacom tablet to map to a specific offset on your screen. This is
something that can be done with the `xsetwacom` tool as well by editing the
`Area` parameter. However, the way `xsetwacom` works is different than the way
wacomconf works. In `xsetwacom`, the `Area` coordinates give the _tablet_
coordinates of the upper left and lower right screen corner. This is highly
unintuitive. Additionally, the default configuration does not honor the aspect
ratio of the tablet at all, which means when you're using an 8:5 tablet and
have a 16:9 screen, the X axis will have a different scale than Y. In other
words, when you draw a physical square, you'll get a rectangle on screen. This
is just awful.

In contrast, wacomconf allows you to set an area in screen coordinates where
you would like the tablet cursor to appear. It also honors the aspect ratio of
the tablet and instead of streching pixels just doesn't allow you to draw
outside of the physical boundaries that the tablet offers.

## Dependencies
Python3, `xdpyinfo` and `xsetwacom`.

## Usage
First, find out your tablet name:

```
$ xsetwacom --list devices
Wacom Bamboo Connect Pen stylus 	id: 10	type: STYLUS
Wacom Bamboo Connect Pen eraser 	id: 11	type: ERASER
Wacom Bamboo Connect Pad pad    	id: 12	type: PAD
```

Note the name of the device that you want to set the options for, in this case
"Wacom Bamboo Connect Pen stylus". Then look at the physical dimensions of the
device, by doing:

```
$ xsetwacom --get "Wacom Bamboo Connect Pen stylus" Area
0 0 14720 9200
```

Note that you'll have to do this before the device has been reconfigured.
You'll see that the X and Y extents are 14720 and 9200. Then, create the
configuration JSON file:

```json
{
    "name":                     "Wacom Bamboo Connect Pen stylus",
    "physical_dimension":       [ 14720, 9200 ],
    "screen_offset_top_left":   [ 1920, 0 ],
    "tablet_width":             1000
}
```

In this case, we want the tablet top left corner to appear at screen
coordinates (1920, 0) and the tablet width to be 1000 screen pixels wide. The
height is automatically determined, preserving the aspect ratio of the tablet.

Then, put the configuration in effect:

```
$ ./wacomconf -vv wacom.json
Screen resolution: 5760 x 1080 (ratio 16 : 3)
Tablet physical aspect ratio: 8 : 5
Tablet offset: Upper left corner at 1920, 0, lower right corner at 2920, 625
Tablet mapping screen area: 1000 x 625 pixels (ratio 8 : 5)
Transformation matrix: Xform<X Linear<14.720 x + -28262.400>, Y Linear<14.720 x + 0.000>>
Mapped upper left of screen in tablet coordinates: -28262, 0
Mapped lower right of screen in tablet coordinates: 56525, 15898
Command: xsetwacom --set 'Wacom Bamboo Connect Pen stylus' Area '-28262 0 56525 15898'
```

You will see that wacomconf has reconfigured the tablet to be from upper left
corner (1920, 0) to (2920, 625). This means the tablet width and height on
screen is 1000 pixels wide (as per the configuration file) and 625 pixels high
(5/8th of 1000 to preserve the aspect ratio).

## License
GNU GPL-3.
