# Scene Description Changes from pbrt-v3 to pbrt-v4

## Table of Contents
* [Preamble](#preamble)
* [Base Scene Description](#base-scene-description-changes)
* [Camera](#camera-changes)
* [Film](#film-changes)
* [Filter](#filter-changes)
* [Integrator](#integrator-changes)
* [Shape](#shape-changes)

## Preamble
This is not meant to be the official document representing the changes between pbrt-v3 and pbrt-v4, I'll leave that to Matt's much more capable hands. These are my notes during the process of updating an exporter from v3 to v4. I am making these public to aid others in the transition until a formal page is created by Matt. The changes listed are deduced from reading through the source code of https://github.com/mmp/pbrt-v4. As pbrt-v4 is still an early release and changing rapidly expect these notes to be out of sync at times. When in doubt, trust the code! :)

### Updating an exporter while the interface is still undergoing changes?
Yup, I'm an idiot. But its exciting to follow the developments and try out the changes!

## Base Scene Description Changes
* `WorldEnd` has been removed
* New `ColorSpace "name"` graphics state Attribute(?)<br>
Sets the ColorSpace to be "name" which defines the color space of spectrum parameters. Accepts the following builtin names -
  * todo
* New `Option "name" "value"` call added<br>
This sets some global options including
  * `bool disablepixeljitter` (false)
  * `bool disablewavelengthjitter` (false)
  * `string msereferenceimage` ("")
  * `string msereferenceout` ("")
  * `integer seed` (0)
  * `bool forcediffuse` (false)
  * `bool pixelstats` (false)
* bool types are no longer quoted
  * pbrt-v3 `"bool parm" ["true"]`
  * pbrt-v4 `"bool parm" [true]`

## Camera Changes
### EnvironmentCamera is now SphericalCamera
* Declaration is now "spherical" (was "environment")
* Add `string mapping` parameter with possible values of
  * equiarea (default)
  * equirect
### PerspectiveCamera
* Remove `float halffov` parameter (just use "fov" parameter)
### RealisticCamera
* Add `float dispersionfactor` parameter (defaults to 0)
* Add `float scale` parameter (defaults to 1)
* Add `string aperture` parameter (defaults to "")<br>
  This parameter supports a image file path or one of the following built-ins
  * gaussian
  * square
  * pentagon
  * star

## Film Changes
### ImageFilm is now RGBFilm
* Declaration is now "rgb" (was "image")
* `xresolution` and `yresolution` defaults have changed to (1280x720) respectively.
* `integer[4] pixelbounds` parameter added. If crop region is specificed it will override this. <br>
(this previously lived on certain integrators)
* `float maxsampleluminance` changed to `float maxcomponentvalue`
* Add `bool savefp16` parameter for saving half images. (default true)
### New GBufferFilm
* Parameters are the same as RGBFilm

## Filter Changes
### All Filters
* Rename `float xwidth` to `float xradius`
* Rename `float ywidth` to `float yradius`
### Gaussian Filter
* `float xradius` and `float yradius` defaults are now 1.5
* `float alpha` is now `float sigma` default is 0.5

## Sampler Changes
### Remove MaxMinDist Sampler
### Remove 02Sequence Sampler
### Sobol Sampler
* Add `string randomization` parameter with the following options
 * none
 * owen (default)
 * cranleypatternson
 * xor
 ### New PaddedSobol Sampler
 * Declared with "paddedsobol"
 * This has the same parameters as the Sobol Sampler
 ### Stratified Sampler
 * Remove `integer dimensions` parameter
 ### Halton Sampler
 * Remove `bool samplepixelcenter` parameter
 ### New PMJ02BN Sampler
 * Declared with "pmj02bn"
 * Has `integer pixelsamples` parameter (defaults to 16)

## Integrator Changes
### General
* `integer[4] pixelbounds` that existed on the various Integrators has been removed and now live on Film
* Default integrator is now  VolumePath
### Remove Whitted Integrator
### Remove DirectLighting Integrator
### Rename AO Integrator
* Type name is now "ambientocclusion", previously it was "ao"
* Remove `integer maxsamples` parameter
* Add `float maxdistance` parameter (defaults to Inf)
### BDPT Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power (default)
  * bvh
  * exhaustive *(new)*
* Add `bool regularize` parameter (defaults to false)
### New LightPath Integrator
* Declared with "lightpath"
* Has `integer maxdepth` parameter (defaults to 5)
### New RandomWalk Integrator
* Declared with "randomwalk"
* Has `integer maxdepth` parameter (defaults to 5)
### MLT Integrator
* Add `bool regularize` parameter (defaults to false)
### Path Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* Add `bool regularize` parameter (defaults to false)
### New SimplePath Integrator
* Declared with "simplepath"
* New parameters include
  * `integer maxdepth` (defaults to 5)
  * `bool samplelights` (defaults to true)
  * `bool samplebsdf` (defaults to true)
### New SimpleVolPath Integrator
* Declared with "simplevolapath"
* Has `integer maxdepth` parameter (defaults to 5)
### SPPM Integrator
* Add `integer seed` parameter (defaults to 0)<br>
(This does **not** lookup the seed value stored in `Options`)
* Add `bool regularize` parameter (defaults to false)
### VolPath Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* Add `bool regularize` parameter (defaults to false)

## Shape Changes
### Cone Shape Removed
### HeightField Shape Removed
### NURBS Shape Removed
### Hyperboloid Shape Removed
### Paraboloid Shape Removed
### Triangle Shape
* Remove `texture alpha` parameter
* Remove `texture shadowalpha` parameter
### Ply Shape
* Remove `texture alpha` parameter
* Remove `texture shadowalpha` parameter

