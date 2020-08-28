# Scene Description Changes from pbrt-v3 to pbrt-v4

## Table of Contents
* [Preamble](#preamble)
* [Base Scene Description](#base-scene-description-changes)
* [Camera](#camera-changes)
* [Film](#film-changes)
* [Filter](#filter-changes)
* [Integrator](#integrator-changes)
* [Light](#light-changes)
* [Material](#material-changes)
* [Texture](#texture-changes)
* [Shape](#shape-changes)
* [Medium](#medium-changes)

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
#### EnvironmentCamera is now SphericalCamera
* Declaration is now "spherical" (was "environment")
* Add `string mapping` parameter with possible values of
  * equiarea (default)
  * equirect
#### PerspectiveCamera
* Remove `float halffov` parameter (just use "fov" parameter)
#### RealisticCamera
* Add `float dispersionfactor` parameter (defaults to 0)
* Add `float scale` parameter (defaults to 1)
* Add `string aperture` parameter (defaults to "")<br>
  This parameter supports a image file path or one of the following built-ins
  * gaussian
  * square
  * pentagon
  * star

## Film Changes
#### ImageFilm is now RGBFilm
* Declaration is now "rgb" (was "image")
* `xresolution` and `yresolution` defaults have changed to (1280x720) respectively.
* `integer[4] pixelbounds` parameter added. If crop region is specificed it will override this. <br>
(this previously lived on certain integrators)
* `float maxsampleluminance` changed to `float maxcomponentvalue`
* Add `bool savefp16` parameter for saving half images. (default true)
#### New GBufferFilm
* Parameters are the same as RGBFilm

## Filter Changes
#### All Filters
* Rename `float xwidth` to `float xradius`
* Rename `float ywidth` to `float yradius`
#### Gaussian Filter
* `float xradius` and `float yradius` defaults are now 1.5
* `float alpha` is now `float sigma` default is 0.5

## Sampler Changes
#### Remove MaxMinDist Sampler
#### Remove 02Sequence Sampler
#### New PMJ02BN Sampler
 * Declared with "pmj02bn"
 * Has `integer pixelsamples` parameter (defaults to 16)
#### New PaddedSobol Sampler
 * Declared with "paddedsobol"
 * This has the same parameters as the Sobol Sampler
#### Sobol Sampler
* Add `string randomization` parameter with the following options
 * none
 * owen (default)
 * cranleypatternson
 * xor
#### Stratified Sampler
 * Remove `integer dimensions` parameter
#### Halton Sampler
 * Remove `bool samplepixelcenter` parameter

## Integrator Changes
#### General
* `integer[4] pixelbounds` that existed on the various Integrators has been removed and now live on Film
* Default integrator is now  VolumePath
#### Remove Whitted Integrator
#### Remove DirectLighting Integrator
#### New LightPath Integrator
* Declared with "lightpath"
* Has `integer maxdepth` parameter (defaults to 5)
#### New RandomWalk Integrator
* Declared with "randomwalk"
* Has `integer maxdepth` parameter (defaults to 5)
#### New SimplePath Integrator
* Declared with "simplepath"
* New parameters include
  * `integer maxdepth` (defaults to 5)
  * `bool samplelights` (defaults to true)
  * `bool samplebsdf` (defaults to true)
#### New SimpleVolPath Integrator
* Declared with "simplevolapath"
* Has `integer maxdepth` parameter (defaults to 5)
#### Rename AO Integrator
* Type name is now "ambientocclusion", previously it was "ao"
* Remove `integer maxsamples` parameter
* Add `float maxdistance` parameter (defaults to Inf)
#### BDPT Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power (default)
  * bvh
  * exhaustive *(new)*
* Add `bool regularize` parameter (defaults to false)
#### MLT Integrator
* Add `bool regularize` parameter (defaults to false)
#### Path Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* Add `bool regularize` parameter (defaults to false)
#### SPPM Integrator
* Add `integer seed` parameter (defaults to 0)<br>
(This does **not** lookup the seed value stored in `Options`)
* Add `bool regularize` parameter (defaults to false)
#### VolPath Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* Add `bool regularize` parameter (defaults to false)

## Light Changes
#### All Lights
* `spectrum scale` is now `float scale`
#### Goniometric Light
* `string  mapname` is now `string filename`
#### Infinite Light
* Add `point[4] portal` parameter
* Remove `integer samples` parameter
* `string  mapname` is now `string filename`
* `spectrum L` and `string filename` are mutually exclusive and should not be declared together.
#### Projection Light
* `string  mapname` is now `string filename`
* Remove `spectrum I` parameter<br>
*Spectrum values come only from the supplied image*
* `float fov` default has changed from 45 to 90
#### Diffuse AreaLight
* Remove `integer samples` parameter
* Add "string filename" parameter (defaults to "")
* `spectrum L` and `string filename` are mutually exclusive and should not be declared together.

## Material Changes
#### General Changes
* `float texture bumpmap` has been renamed to `float texture displacement`
#### Following Materials Have Been Removed
* Uber
* Translucent
* Substrate
* Plastic
* Mirror
* Metal
* Matte
* KdSubsurface
* Glass
* Fourier
* Disney
#### CoatedConductor Material
Type name: "coatedconductor"
##### Parameters:
* displacement (float texture)
* interface.eta (float texture)
* thickness (float texture)
* interface.roughness (float texture)
* interface.uroughness (float texture)
* interface.vroughness (float texture)
* conductor.eta (spectrum texture)
* conductor.k (spectrum texture)
* conductor.roughness (float texture)
* conductor.uroughness (float texture)
* conductor.vroughness (float texture)
* remaproughness (bool)
* maxdepth (integer)
* nsamples (integer)
#### CoatedDiffuse Material
Type name: "coateddiffuse"
##### Parameters:
* displacement (float texture)
* reflectance (spectrum texture)
* eta (float texture)
* thickness (float texture)
* roughness (float texture)
* uroughness (float texture)
* vroughness (float texture)
* remaproughness (bool)
* maxdepth (integer)
* nsamples (integer)
* twosided (bool)
#### Conductor Material
Type name: "conductor"
##### Parameters:
* displacement (float texture)
* eta (spectrum texture)
* k (spectrum texture)
* roughness (float texture)
* uroughness (float texture)
* vroughness (float texture)
* remaproughness (bool)
#### Diffuse Material
Type name: "diffuse"
##### Parameters:
* displacement (float texture)
* reflectance (spectrum texture)
* sigma (float texture)
#### DiffuseTransmission Material
Type name: "diffusetransmission"
##### Parameters:
* displacement (float texture)
* reflectance (spectrum texture)
* transmission (spectrum texture)
* sigma (float texture)
* scale (float)
#### Measured Material
Type name: "measured"
##### Parameters:
* displacement (float texture)
* filename (string)
#### ThinDielectric Material
Type name: "thindielectric"
##### Parameters:
* displacement (float texture)
* eta (float/spectrum texture)
#### Dielectric Material
Type name: "dielectric"
##### Parameters:
* displacement (float texture)
* eta (float/spectrum texture)
* roughness (float texture)
* uroughness (float texture)
* vroughness (float texture)
* remaproughness (bool)
#### Subsurface Material
* Remove `spectrum Kr` parameter
* Remove `spectrum Kt` parameter
* Add `spectrum texture reflectance` parameter
* Add `spectrum texture mfp` parameter
* Add `float g` parameter
* Add `float texture roughness` parameter
#### Mix Material
* Change `spectrum texture amount` to `float texture amount`

## Texture Changes
#### Remove UVTexture
#### Fbm Texture
* Output is now only float, in pbrt-v3 it was float and spectrum.
#### ImageMap Texture
* Add `float scale` parmeter, (defaults to 1)
* Add new "octahedralsphere" mode to `string wrap`
* Remove `bool trilinear` parameter
* Add `string filter` parametr, options include
  * point
  * bilinear (default)
  * trilinear
  * ewa
* Remove `bool gamma` parameter
* Add `string encoding` parameter, this accepts either a builtin name or a "gamma value".<br>
One of the following options can be specified. (The default is based on the file extension of the texture.)
  * linear
  * sRGB
  * gamma float_value
#### Marble Texture
* Output is now only spectrum, in pbrt-v3 it was float and spectrum
#### Scale Texture
* `texture tex1` is now just `texture tex`
* `texture tex2` has been replaced with `float scale` (defaults to 1)
#### Windy Texture
* Output is now only float, in pbrt-v3 it was float and spectrum.
#### Wrinkled Texture
* Output is now only float, in pbrt-v3 it was float and spectrum.

## Shape Changes
#### General
All shapes now support an `float texture alpha` parameter.
#### Shape Removals
* Cone
* HeightField
* NURBS
* Hyperboloid
* Paraboloid
#### New Bilinear Patch Mesh
* Declared with "bilinearmesh"
* Parameters:
  * `point3[] P`
  * `integer[] indices`
  * `point2[] uv` optional
  * `normal3[] N` optional
  * `integer[] faceIndices` optional
  * `string emissionfilename` optional
#### Triangle Shape
* Remove `texture shadowalpha` parameter
* Change uv parameter from `float[] uv` to `point2[] uv`
#### Ply Shape
* Remove `texture shadowalpha` parameter

## Medium Changes
#### Homogeneous Grid
* New `spectrum Le` parameter
#### General Mediums
Mediums with a varying medium now have a common interface to define scattering properties. These parameters include:
* `string preset` default ("")<br>
*The list of presets scattering properties has not changed since pbrt-v3*
* `spectrum sigma_a` defaults to (0.0011, 0.0024, 0.014)
* `spectrum sigma_s` defaults to (2.55, 3.21, 3.77)
* `float scale` defaults to 1.0
* `float g` defaults to 0.0
#### New Uniform Medium
This was previously the Heterogeneous Medium in pbrt-v3
* Declare with "uniformgrid"
* Parameters include:
  * General Medium parameters described above
  * `float[]|rgb[] density`
  * `integer nx` defaults to 1
  * `integer ny` defaults to 1
  * `integer nz` defaults to 1
  * `spectrum Le` defaults to 0
  * `float[] Lescale`
  * `point3 p0` defaults to (0,0,0)
  * `point3 p1` defaults to (1,1,1)
#### New Cloud Medium
* Declare with "cloud"
* Parameters include:
  * General Medium parameters described above
  * `float density` defaults to 1
  * `float wispiness` defaults to 1
  * `float extent` defauts to 1
  * `point3 p0` defaults to (0,0,0)
  * `point3 p1` defaults to (1,1,1)  
#### New NanoVDB Medium
* Declare with "nanovdb"<br>
Looks for fields within the VDB of the name "density" and "temperature"
* Parameters include:
  * General Medium parameters described above
  * `string filename` defaults to ""
  * `float LeScale` defaults to 1.0
  * `float temperaturecutoff` defaults to 0.0
  * `float temperaturescale` defaults to 1.0
 
