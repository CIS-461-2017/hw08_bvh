Path Tracer Episode VI: Return of the Acceleration Structures
======================

**University of Pennsylvania, CIS 561: Advanced Computer Graphics, Homework 8**

Overview
------------
Thus far, we have been rendering scenes with relatively little geometry.
However, if we want to be able to produce more interesting images, we'll need
to be able to render scenes with large numbers of polygons, e.g. triangle
meshes, without taking forever. To that end, we will implement a Bounding
Volume Hierarchy to contain all Primitives in the scene so that we can perform
ray-object intersection more efficiently.

Once again, we have provided updated files. Most notably, we have altered the
`Mesh` class so that it exists only to render triangles in the OpenGL preview.
Now, individual `Triangle`s are instantiated as `Primitive`s and added to the
`Scene` as such. This should simplify your BVH construction.

Useful Reading
---------
We recommend referring to the following PBRT chapters when implementing this
assignment:
* Chapter 4.2: Aggregates
* Chapter 4.3: Bounding Volume Hierarchies

The Light Transport Equation
--------------
#### L<sub>o</sub>(p, &#969;<sub>o</sub>) = L<sub>e</sub>(p, &#969;<sub>o</sub>) + &#8747;<sub><sub>S</sub></sub> f(p, &#969;<sub>o</sub>, &#969;<sub>i</sub>) L<sub>i</sub>(p, &#969;<sub>i</sub>) V(p', p) |dot(&#969;<sub>i</sub>, N)| _d_&#969;<sub>i</sub>

* __L<sub>o</sub>__ is the light that exits point _p_ along ray &#969;<sub>o</sub>.
* __L<sub>e</sub>__ is the light inherently emitted by the surface at point _p_
along ray &#969;<sub>o</sub>.
* __&#8747;<sub><sub>S</sub></sub>__ is the integral over the sphere of ray
directions from which light can reach point _p_. &#969;<sub>o</sub> and
&#969;<sub>i</sub> are within this domain.
* __f__ is the Bidirectional Scattering Distribution Function of the material at
point _p_, which evaluates the proportion of energy received from
&#969;<sub>i</sub> at point _p_ that is reflected along &#969;<sub>o</sub>.
* __L<sub>i</sub>__ is the light energy that reaches point _p_ from the ray
&#969;<sub>i</sub>. This is the recursive term of the LTE.
* __V__ is a simple visibility test that determines if the surface point _p_' from
which &#969;<sub>i</sub> originates is visible to _p_. It returns 1 if there is
no obstruction, and 0 is there is something between _p_ and _p_'. This is really
only included in the LTE when one generates &#969;<sub>i</sub> by randomly
choosing a point of origin in the scene rather than generating a ray and finding
its intersection with the scene.
* The __absolute-value dot product__ term accounts for Lambert's Law of Cosines.

Updating this README (5 points)
-------------
Make sure that you fill out this `README.md` file with your name and PennKey,
along with your test renders. You should render the new scenes we have provided
you, and for at least PT_wahoo.json compare the render time with and without the
BVH acceleration structure. Note that PT_wahoo.json will take an incredibly long
time to render without an acceleration structure if you use the default render
settings, so we recommend rendering it with as few as 2x2 samples per pixel
with the `DirectLightingIntegrator` in order to compare results.

General Requirements
--------------
For this assignment, all we ask is that you implement a BVH acceleration
structure. Whether or not you use PBRT's full implementation is up to you, but
we encourage you to design the BVH your own way. PBRT's structure is highly
optimized, and their code is particularly opaque and difficult to debug, and
you will benefit more as a coder if you try to translate the BVH concept into
code. All we ask is that your program and BVH fulfill the requirements below.

`Bounds3f` class functions (20 points)
----------
We have provided you a 3D axis-aligned bounding box class `Bounds3f` in
`scene/bounds.h`. Please implement the three functions that have been
declared at the bottom of the class: `Apply`, `SurfaceArea`, and `Intersect`.
* `Apply` should transform the eight corners of the bounding box by the input
`Transform`'s matrix, then compute a new axis-aligned bounding box that
encompasses these corners (which may no longer be axis aligned, especially if
the `Transform` contained a rotation). This function sets the invoking
`Bounds3f` to be this bounding box, and returns the resultant bounds.
* `SurfaceArea`, as its name implies, computes and returns the surface area of
the invoking `Bounds3f`. This will be used when you apply the surface area
heuristic to choose a place at which to split yourBVH.
* `Intersect` computes the intersection of the input ray with this bounding box.
Unlike the `Intersect` functions of the `Shape` classes, this does not output
an entire `Intersection` data package, but just outputs the `t` parameterization
that the intersection would have with the ray. This is useful for determining
which bounding box is intersected first by a given ray. Additionally, negative
`t` values are valid _if and only if_ the ray's origin lies within the bounding
box.

In addition to these functions, we have already provided you with a few
utility functions that you may or may not use, such as `MaximumExtent`, which
returns the index of the axis along which the bounding box has the largest side
length, or `Union`, which creates a `Bounds3f` that encompasses all inputs.

`Shape` class `WorldBound` functions (10 points)
-----------
We have updated the `Shape` class to include a `WorldBound` function, which
computes the axis-aligned bounding box of the shape after it has been
transformed into world space. Please implement this function for each of the
classes which inherit from the `Shape` class. You will find `Bounds3f`'s `Apply`
function useful when implementing all of the various `Shape`s' `WorldBound`
functions. You will need to compute the world-space bounding boxes of all your
scenes' `Primitives` in order to construct your BVH.

`BVHAccel` class (50 points)
-------
We have provided you with a class that represents a bounding volume hierarchy,
`BVHAccel`. Most of the details of implementation are left for you to decide,
but you will need to implement at least two of the functions we have provided:
`BVHAccel`'s constructor, and its `Intersect` function. The constructor is where
you should build the hierarchy of bounding box nodes; the only detail we require
of this construction is that you use the Surface Area Heuristic to determine
where to split your geometry. We have provided you with the intermediate
`struct`s that PBRT uses in its implementation of its BVH, but we do not require
you to use any of them.

A branch of BVH construction should produce a leaf node when there would be
at most `maxPrimsInNode` `Primitives` within that node, or when the cost of
subdividing that node via the surface area heuristic would be greater than the
cost of simply testing for intersection against all primitives in that node.
Remember that the SAH cost of a subdivision is estimated as
`(surface area of left box * number of primitives in left box + surface area of
right box * number of primitives in right box ) / (surface area of encompassing
box)`. Likewise the cost of intersecting a node is simply `number of primitives
in the node`.

Timing your BVH's construction (5 points)
--------------
Use a `QTime` to measure how many milliseconds it takes your BVH to be
constructed. `QTime::start()` sets the `QTime`'s internal time counter to the
present time, and `QTime::elapsed()` returns how many milliseconds have gone by
since `start()` was invoked. Your program should print this time via `cout`,
and there should be some sort of print message labeling this number as the
build time of the BVH.

Please record this time in your README along with your render times and renders.

Custom scene (10 points)
-----------
Now that you have implemented an acceleration structure, we'd like you to take
advantage of it to render a scene with a large number of polygons, ideally a
triangle mesh. Create a custom scene that includes an interesting mesh;
[PBRT's web site](http://www.pbrt.org/scenes-v3.html) has many OBJs free for
download. Note that the scenes included in that link are in a format that
our JSON parser does not read; you'll have to use the OBJs only. As always, we
encourage high sample counts and a large image resolution, but only require
400 samples per pixel and 512x512 resolution. Additionally, please remember to
include your scene file JSON and any dependencies, e.g. textures or OBJs, that
would be required to render it.

Extra credit (30 points maximum)
-----------
As always, you are free to implement other features for extra credit that are
not listed below. Make a private post on Piazza so we can help you determine how
many points your feature(s) would be worth.

#### Parallelizing your BVH construction (20 points)
Code the construction of your BVH such that it can be easily performed on
multiple CPU cores simultaneously, and leverage a multithreading library (Qt's
multithreading classes are the most straightforward to use in our case) to
make your BVH construction run in parallel. The scene's BVH is constructed in
a call made within `MyGL::RenderScene`. Compare the construction time of your
BVH with and without multithreading (you don't need to make your BVH capabale
of changing construction type in your final submission; just record a
construction time once before you multithread your BVH).

#### k-d Tree (20 points)
Implement the construction of a k-d tree in addition to your BVH. The primary
difference between these two structures is that a k-d tree divides up space
rather than primitives. The process of intersecting a ray with either tree is
fairly similar. Refer to the lecture slides and chapter 4.4 of the textbook for
more information.

Submitting your project
--------------
Along with your project code, make sure that you fill out this `README.md` file
with your name and PennKey, along with your test renders.

Rather than uploading a zip file to Canvas, you will simply submit a link to
the committed version of your code you wish us to grade. If you click on the
__Commits__ tab of your repository on Github, you will be brought to a list of
commits you've made. Simply click on the one you wish for us to grade, then copy
and paste the URL of the page into the Canvas submission form.
