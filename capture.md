Screenshot playback in editors
==============================

In the _3D Viewport_, you can go to the _View_ menu and select _Viewport Render Animation_ and capture what the camera sees in that viewport.

Oddly, the same feature isn't available for all editors.

So, the following describes how to achieve something similar using the Python API.

Python console
--------------

Open a _Python Console_ and an _Info_ area. Change the editor that you're interested in recording to some other random editor and then switch it back, in the _Info_ area, the last row should be something like:

```
bpy.context.area.ui_type = 'CLIP_EDITOR'
```

Now, you know the `ui_type` to specify below, i.e. in this case `CLIP_EDITOR`.

Note: you'll need extra conditions, e.g. `area.spaces[0].view == 'CLIP'` as below, if you've got different instances of the same editor open.

First, get the relevant editor set up exactly as you want it, e.g. press `n` and `t` to show/hide the left and right sidebars.

Make sure you're on the last frame, i.e. `shift-right-arrow`.

Question: why the _last_ frame? Answer: because below you first jump to the first frame (so you get it) and then press play (which will get the second frame and those that follow).

Paste the following into the _Python Console_, change `/tmp` if you want the screenshots saved elsewhere:

```
area = next(area for area in C.screen.areas if area.ui_type == 'CLIP_EDITOR' and area.spaces[0].view == 'CLIP')

def save_screenshot(scene):
    with C.temp_override(area=area):
        bpy.ops.screen.screenshot_area(filepath=f'/tmp/{scene.frame_current:04}.png', check_existing=False)

def stop_playback(scene):
    if (scene.frame_current == scene.frame_end):
        bpy.ops.screen.animation_cancel(restore_frame=False)
        
bpy.app.handlers.frame_change_post.append(save_screenshot)
bpy.app.handlers.frame_change_post.append(stop_playback)
```

Then press `shift-left-click` (i.e. jump to first frame) and then press _Play_, i.e. `spacebar`, and wait for the playback to finish.

Note: the `stop_playback` handler is needed above as there's no other way to prevent playback looping (it's a long requested feature, see [here](https://blender.community/c/rightclickselect/nKfbbc)).

Fixing off-by-one problem
-------------------------

Now, the weird bit - depending on the editor involved, you either capture the expected frame number or the one before, e.g. when taking screen shots of the _Playback_ editor everything worked as expected but when taking screen shots of the _Tracking_ editor, I see the following:

* Jumping from the last frame to the first frame results in the last frame ending up in `0001.png`.
* And once I start playback, this continues, i.e. frame 1 ends up in `0002.png` and so on.
* And then at the end things go even more wrong, frame 318 end up in frame `0319.png` (following the existing pattern) but then frame 320 (the last frame) ends up in `0320.png`.

I.e. the last frame is the only one that ends up in a file with the same number but we end up losing the second last frame, i.e. 319, and frames `0001.png` and `0320.png` are the same.

So, to fix this, I first removed the `save_screenshot` handler:

```
bpy.app.handlers.frame_change_post.remove(save_screenshot)
```

Then cleaned things up in `bash`:

```
$ mkdir fixed
$ mv /tmp/????.png fixed
$ cd fixed
$ rm 0001.png 
$ mv 0320.png 0321.png
```

And back in Blender moved to frame 319, i.e. the missing penultimate frame, and then selected _Window / Save Screenshot (Editor)_, clicked the _Tracking_ editor and then saved out the screenshot as `0320.png` (i.e. still one more than its actual frame number).

And then corrected for everything being off by one in `bash`:

```
$ for i in {1..320}
do
    printf -v padA "%04d" $i
    printf -v padB "%04d" $((i + 1))
    mv $padB.png $padA.png
done
```

Cropping the screenshots
------------------------

Then use Gimp to find the upper-left-most pixel that you want - hover over the actual pixel you want to include (and not e.g. the one just above and to the right) and you'll see it's location bottom right, e.g. `(103, 390)`.

And drag out a rectangular selection to find the width and height you want, e.g. I'd used `Numpad 2` to get a fractional zoom of 1:2 for my 4k clip so as expected the resulting rectangle was 1920x1080.

Then use these values to crop out this area with ImageMagick:

```
$ convert 0001.png -crop 1920x1080+103+273 crop.png
```

To crop every file (remove `crop.png` first or it'll be double cropped):

```
$ mkdir cropped
$ mogrify -crop 1920x1080+103+273 -path cropped *.png
```
