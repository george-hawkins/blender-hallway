Hallway
=======

The results are in `render-results` - e.g. `render-results/01-tracking` contains the frames for showing the tracking and `render-results/01-tracking.mkv` is the corresponding video file.

For making a viewport animation for an editor other than the _3D Viewport_, see [`capture.md`](capture.md).

For breakdown of segments that make up the overall clip, see [here](sounds/README.md).

For a lo-res lo-quality render of the clip without sound, see [here](render-results/00-final/lo-res.mp4)

Viewport rendering
------------------

When using _View / Viewport Render ..._, remember to turn off _Transparent_ or you get a pure black background.

Files
-----

* Movie clip `.png` frames - `frames-4k`
* Masked near and far `.exr` frames - `frames-masked-near` - `frames-masked-far`
* Moving out the far door `.png` frames - `render-animated-door`

* `textures` for `castle_wall_slates` and `cobblestone_floor_04`

* Stabilized scary door - `scary-door-4k-70-stable-250-frames.mkv`
* Moving out the far door - `animate-far-door.mp4`


* `animate-far-door.blend` - first, go to the _Layout_ tab first and play the animation - all that's happening is that the near wall is staying as it is on frame 201 and the far wall is being pushed out. Then go to the _Compositing_ tab to see how the 201 frame and the animated far wall (along with the floor) are combined. If you go to _Motion Tracking_ tab, things get a bit confusing - we seem to have imported the stabilization data but I'm not sure what's being stabilized - just frame 201 or also the door as it move out?
* `sequencer-animated-door.blend` - simply the sequencer for moving the far door out.

* `hallway-stab-201-near-far.blend` - I'm not sure what's going on here, I suspect the _Transform_ node was the key element here.
* `hallway-stabilization.blend` - ditto, the fact that it's in `archive` (see below) suggests it was a finished _something_.

* `hubble-leo-triplet-ng.blend` - has no geometry, combines near and far `.exr` frames with star field and stabilization. As it's in `archive`, I guess it's considered finished.

`archive` seems to be a backup for `.blend` files that were viewed as completed - to use them you have to copy them up to the parent directory or else all relative paths are wrong.

* `hallway-mask.blend` - masks for near and far door.
* `hallway-ng-2.blend` - hallway with no right wall.
* `hallway9.blend` - hallway with right wall and skirting board.
* `hallway-build.blend` - building up the hallway.


* `build-hallway-5-pig-king.blend` - pig king hallway.
* `hallway-narrow.blend` - final narrow cobbled street.
* `hallway-widening-animation.blend` - streed widening from narrow to wide (from frame 280).
* `hallway-widening-animation-206.blend` - streed widening from narrow to wide (from frame 206).
* `hallway-wide.blend` - wide cobbled street.
* `hallway-wide-and-bug-5-reverse.blend` - wide cobbled street with bug.
* `hallway-sci-fi.blend` - sci-fi exterior.
* `hubble-leo-triplet-eevee.blend` - hubble exterior (doesn't need Cycles).

Note: only `animate-far-door.blend` above has a wide floor that "slides" under the near door.

For a good overview of view layers, see: https://youtu.be/PByGtB_ahmw

Subtitling
----------

<https://www.checksub.com/subtitle/do-good-subtitles-golden-rules/> says 20 characters per second with a maximum of two lines of at most 40 characters each.

<https://www.unimelb.edu.au/accessibility/video-captioning/style-guide> says 180 words per minute, i.e. 3 words per second. And no caption should appear for less than 2 seconds.

Eventually, I settled on 2&frac23; words per second and no caption should appear for less than 2 seconds or more than 8 seconds.

Masks
-----

When I talk about masks here, I'm not talking about the Blender [masks concept](https://docs.blender.org/manual/en/3.1/movie_clip/masking/introduction.html) - I'm just talking about simple image sequences created using geometry to mask out parts of my movie clip.

### Near door

![img.png](images/near-door-mask.png)

The door isn't quite as simple as it seems - it's two large rectangles joined at the top edge, one corresponds to the _outer_ edge of the door and the other one is like a sheet drawn down from the top edge to the outer edge of wooden threshold that extends beyond the door frame. Then these two large rectangles are joined using narrower ones at the sides and the bottom (the bottom one isn't really necessary).

The material is set up as shown above so that the front of the faces is pure black and that back a [holdout](https://docs.blender.org/manual/en/3.1/render/shader_nodes/shader/holdout.html). The holdout ensures we don't see a face when looking at it from the "wrong" side.

The mask does not have to be black. In fact, one could leave the default _Principled BSDF_ - all that matters is that it has no alpha. The advantage of using an _RGB_ node is that it renders very fast and its clear it's a mask rather than scene geometry.

| Rendered | Layout |
|----------|--------|
| ![img.png](images/near-door-rendered.png) | ![img.png](images/near-door-layout.png) |

### Far door

The far door is far more modelled than the near one (wildly over modelled if you look at Ian Hubert's [videos](https://www.youtube.com/c/mrdodobird/videos) and see how the eye is willing to put up with simple geometry if you've got good textures).

![img.png](images/far-door-mask.png)

Mask composition
----------------

![img.png](images/mask-composition.png)

First, I hide everything for both the viewport and renders except for the near door mask object and the camera and (using the node setup above) rendered out my near layer.

Then I hid the near door mask object and unhid the far one, muted the invert node and rendered out my far layer.

TODO: add in explanation below of why you used OpenEXR.

Note: when rendering out the near layer, something odd happened on the last ten or so frames - the camera essentially moved through the mask object and so saw the full movie frame behind it. I don't know why this happened as, looking at the scene from other angles, the camera still seemed to be clearly in front of the mask object. As the frames were now just supposed to be all alpha, I just copied earlier frames onto the problematic ones and didn't investigate.

Naming your movie clip
----------------------

I'd never noticed this before but you can name your movie clip. This becomes more relevant if you want to pull the data associated with your camera tracking and stabilization into another `.blend` file (via [_Append_](https://docs.blender.org/manual/en/3.1/files/linked_libraries/link_append.html)) and want things clearly named.

By default, the clip name is just the name of the first frame that you import (this is less odd when working with videos, rather than frames, where the name would be something like `hallway.mkv`). I.e. the `0001.png` shown here in the top bar of the _Movie Clip_ editor:

![img.png](images/movie-clip-name.png)

You can change it to something more meaningful, e.g. "Hallway Clip" and then find it (or rather its associated tracking and stabilization data) more easily later when this will be the name of the relevant data blocks.

Stabilization
-------------

Once you've gone to the trouble on tracking your scene, you might as well use this high quality data to stabilize the scene rather than e.g. leave it to e.g. the Adobe [Warp Stabilizer](https://helpx.adobe.com/premiere-pro/using/stabilize-motion-warp-stabilizer-effect.html) or DaVinci Resolve [stabilization](https://motionarray.com/learn/davinci-resolve/davinci-resolve-stabilize/) or iMovie's simple [stabilize shaky video](https://support.apple.com/guide/imovie/stabilize-shaky-clips-mov52b5a78a6/mac).

Other stabilizers may have to guess their trackers but do more to the image than the translation, rotation and scale that Blender does - the Warp in the Warp Stabilizer name gives this away. Whatever you do, you cannot recover what the camera would have actually seen, i.e. you can't recover the parallax changes that would have resulted if the original camera was moved around in the same way the stabilizer software moves the frames around.

As the stabilizer can't recover the parallax changes, stabilization can only ever be a post-processing step, i.e. you can't stabilize the solved camera motion with this data, you can only stabilize the resulting renders. Though, there are all kinds of _tricks_ that you can apply, e.g. as discussed by Ian Hubert in his "Wild Tricks for Greenscreen in Blender" [video](https://www.youtube.com/watch?v=RxD6H3ri8RI) but they are tricks.

### Process

Stabilization turned out to be super simple. The only issue I had was with animating target x and y and with autoscale.

I chose two tracking point for which I wanted the location and rotation to stay constant:

![img.png](images/stabilization-trackers.png)

Then I went to the stabilization tab, ticked _2D Stabilization_ and then _Rotation_ and added the two tracks to the _Location_ and _Rotation_ lists (you only really need one for location):

![img.png](images/stabilization-tab-2.png)

And then turned on _Show Stable_:

![img_1.png](images/show-stable-3.png)

_Autoscale_ should have been great but I found it scaled up way too much, if you tick it, you see _Expected Scale_ greyed out but set to the value that _Autoscale_ chooses but I found you could bring the scale much lower than the chosen value. In the end I left _Autoscale_ off and just scrubbed through the video and each time I saw black edges, I nudged the scale up enough to eliminate them.

The amount of black outline could be greatly reduced by animating the location, i.e. not requiring the trackers stay in exactly the same location throughout the clip. On frame 1, right-click _Target X_ and select _Insert Keyframe_, then jump to the end and adjust _Target X_ and _Target Y_ until there's as little black edges as possible and then right-click again and add another keyframe.

Then scrub though the clip and find the frames with the fattest black edges and nudge up the _Expected Scale_ to eliminate them. You can also add additional keyframes on the targe X and Y values and on the expected scale.

Note: adding keyframes and animating the _Target X_ value is how you'd deal with a left-to-right pan - otherwise your image will drift off the side due to the locked location of the tracks. For more on this and "dancing black borders", see the Blender [introduction](https://docs.blender.org/manual/en/3.1/movie_clip/tracking/clip/sidebar/stabilization/introduction.html) to its 2D stabilization documentation.

---

In the end I lowered the location and rotation influence to 0.7 and set expected scale to 1.05 and then nudged it up (and just adjusted the location of the final frame) until I got no black edges - for my clip I ended up nudging it to 1.07.

---

I haven't thought about this deeply - but I suspect you always want to lock the location of the back-most element. If you lock the location of a foreground element and let the background move then I suspect you're making things maximally visually disturbing. You're simulating the effect on a boat where what's near (the structure of the surrounding boat) stays relatively stable while thing on the horizon move more - the opposite to the normal situation and something that contributes to seasickness.

Scary door
----------

Note: initially, I rendered out scary-door with filmic enabled (with the expected washed out result). As always, don't forget to switch to standard when dealing with basic sRGB data from cameras etc!

![img.png](images/scary-door-compositing.png)

TODO: explain that it's here that the difference between premultiplied and straight alpha comes into play. The _Alpha Over_ nodes (and everything else in Blender) expect premultiplied alpha. You can correct for straight alpha with _Convert Premultiplied_ but it becomes a bit confusing when combining multiple sources and you have to look very carefully for the lack of jagged edges to make sure you've gotten things right.

See the Blender glossary item for [alpha channel](https://docs.blender.org/manual/en/3.1/glossary/index.html#term-Alpha-Channel) for a good explanation of the different alpha methods and the issues with converting between them.

Above, we have to two rendered out layers, corresponding to near and far, and an extremely simple scene (seen in the _Render Layers_ node).

The scene just consists of an image that's been imported as a plane (with the standard _Images as Planes_ [addon](https://docs.blender.org/manual/en/3.1/addons/import_export/images_as_planes.html)) and a camera (the tiny orange dot on the right). The image has been scaled up to 100m so that the relative motion of the camera is very small in comparison, i.e. so that the supposedly huge galaxy doesn't move around too much as the camera moves around.

![img.png](images/galaxy-scene.png)

As I wanted the galaxy image to render exactly as it is, I got rid on the associated shader and plugged the image straight into the _Material Output_:

![img.png](images/galaxy-material.png)

The picture of M 66 (one of the galaxies in the Leo Triplet) comes from [here](https://esahubble.org/images/heic1006a/) on ESA Hubble site.

Importing solver and stabilization data
---------------------------------------

I constructed the galaxy `.blend` file from scratch so, to get the camera motion that I'd solved for previously, I needed to be able to import this. Initially, this was a bit confusing as it's not obvious where this data is. The _Movie Clip_ node in the compositor sounds quite generic but actually it's implicitly tied to data behind the camera solve and the resulting camera constraint.

Aside: the nodes _Movie Clip_ and _Image_ seem confusingly named. The _Movie Clip_ node is really for use where you've got tracking data and the _Image_ node is the less specific node (and can handle still images, image sequences and video clips).

As noted up above, in your original tracking `.blend` file, you can name the data block containing all the relevant data via the top bar of the _Movie Clip_ editor. So to import this data and use it for the constraint on a camera in a new `.blend` file:

**1.** First, go to _File / Append_, find the original `.blend` file, open it and navigate into its _MovieClip_ subfolder, select the data block (that you named as just described) and press _Append_:

![img.png](images/append-movie-clip-data-block.png)

**2.** Now, select the camera, to which you want add the solved motion, go to its _Object Constraint_ properties, add a _Camera Solver_ constraint and untick the _Active Clip_ checkbox and instead select the data block, that you just imported, from the _Movie Clip_ dropdown:

![img.png](images/camera-solver-imported.png)

If you look at the camera, you'll now see it move as you drag the play head.

### Stabilization

The imported _MovieClip_ data block also contains any 2D stabilization data that you created. To use this, just add in a _Stabilize 2D_ node as the last node in your _Compositing_ workflow:

![img.png](images/stabilize-2d-node.png)

And rather than click the _Open_ button, click the clapper-board dropdown and select the imported _MovieClip_ data block.

**Important:** don't forget to flip from _Bilinear_ to _Bicubic_ for higher quality interpolation.

Aside: Blender defaults to low-quality bilinear interpolation in various situations, e.g. don't use the _Scale_ node that always uses bilinear interpolation - instead, use the _Transform_ node where you can specify that you want to use bicubic interpolation.

Pushing out the door
--------------------

The stabilization caused an issue for pushing out the door. Ideally, I'd have worked out the translation and scale applied by the stabilization at the frame at which I wanted to push out the door and then converted the _Stabilize 2D_ node to a _Transform_ node that did the same. But I couldn't work out how to get the translation and scale values 

---

Things I learned - that you can plug a color or image straight into material output (?) without a Principled BSDF (?).

Question:

Why did I need to tick "Convert Premultiplied" for the images I rendered out for near and far. Could I have rendered them out with premultiplied alpha?

To "append" the camera constraint, the thing was to copy the _MovieClip_ resource (and not the camera itself), then I could add a _Camera Solver_ constraint to my camera and untick _Active Clip_ and select the imported _MovieClip_.

Alt and MMB can drag around compositing backdrop.

I experienced something really weird - when rendering out masked near frames, frame 0234 rendered with a weird alpha. When I regenerated the same frame (I did 233 to 235), it rendered fine - see 0234-good.png and 0234-bad.png - try flattening both in Gimp, the bad one contains a ghost of the underlying image.

I learned that there's premultiplied alpha and straight alpha and Blender defaults to expecting premultiplied alpha.

It turns out that PNG only supports straight alpha.

It seems Blender even incorrectly supported premultiplied alpha with PNG but have fixed this. Much to many people's annoyance - see <https://developer.blender.org/T81390>

So you have to use a file format that supports premultiplied alpha (or correct it later by with "Convert Premultiplied").

I tried various formats and most of them produce much larger files than PNG. The best were TIFF and OpenEXR (but only when using the lossy DWAB codec) both of them default to full-float color depth and nothing is gained by switching to half-float for TIFF and almost nothing for OpenEXR (presumably their compression algorithms see there's little or no difference in the underlying data and nothing's gained by going for half). Interestingly, PNG though does show a significant difference between 16 and 8 bits.

Comparison:

| Bytes | File |
|-------|------|
| 1733956 | dwab-0001.exr |
| 2323232 | 8-bit-slow-0001.png |
| 2804890 | 8-bit-0001.png |
| 3252828 | 0001.tif |
| 3360855 | 16-bit-0001.png |

So DWAB does the best by far. 8-bit PNG with noticeably slow maximum compression is next, then 8-bit PNG (with the default Blender PNG compression setting), then full-float TIFF and finally 16 bit PNG.

Everything else is substantially larger.

Using DWAB or TIFF resulted in (to me) perceptually identical results and both used premultiplied alpha (which was the goal).

TIFF is lossless but he DWAB files are almost 50% smaller so I went with it. TIFF though is nicer to work with - tools like `eog` can handle it.

---

Or just use a _Converter / Alpha Convert_ node in the _Compositing_ workspace.

---

Removing the background _Render Layers_ node in the _Compositing_ workspace isn't enough to stop it doing at least some work rendering the background - you have to actively delete the background as a _View Layer_.

---

Tracking:

Show search - alt-S

Lock track - ctrl-L

Remember to set tracking to stop 8px before hitting the edge of the image.

---

If you refine for lens distortion, remember that this will affect proxies etc.

Setup tracking scene will add the correct composition nodes to apply the distortion correction values found by refining.

---

You can rename the _Movie Clips_ data block in the top bar of the _Movie Clip_ editor, e.g. change `0001.png` to `my-hallway`.

---

Replacing the _Principles BS??_ with black isn't so important - what we're interested in is the see thru bits. Anything with _any_ color will not be see thru. But replacing the shader results in something that renders faster.

---
