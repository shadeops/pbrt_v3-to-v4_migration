# Scene Description Changes from pbrt-v3 to pbrt-v4

## Table of Contents
* [Preamble](#preamble)
* [Base Scene Description](#base-scene-description-changes)
* [Camera](#camera-changes)
* [Film](#film-changes)
* [Filter](#filter-changes)

## Preamble
This is not meant to be the official document representing the changes between pbrt-v3 and pbrt-v4, I'll leave that to Matt's much more capable hands. These are my notes during the process of updating an exporter from v3 to v4. I am making these public to aid others in the transition until a formal page is created by Matt. The changes listed are deduced from reading through the source code of https://github.com/mmp/pbrt-v4. As pbrt-v4 is still an early release and changing rapidly expect these notes to be out of sync at times. When in doubt, trust the code! :)

### Updating an exporter while the interface is still undergoing changes?
Yup, I'm an idiot. But its exciting to follow the developments and try out the changes!

## Base Scene Description Changes
* `WorldEnd` has been removed
* New `ColorSpace "name"` graphics state Attribute(?)<br>
Sets the ColorSpace to be "name" which defines the color space of spectrum parameters. Accepts the following builtin names -
 * todo
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
 
