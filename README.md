# Scene Description Changes from pbrt-v3 to pbrt-v4

This list is currently based off the changes made up to this commit- https://github.com/mmp/pbrt-v4/commit/4473987 

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

### Legend
A few emoji are used to call attention for the reasons listed below-
| Icon | Reasoning |
|----|----|
|:exclamation: | Significant change to scene description structure |

## Base Scene Description Changes
* :exclamation:`WorldEnd` has been removed
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
* :exclamation: bool types are no longer quoted
  * pbrt-v3 `"bool parm" ["true"]`
  * pbrt-v4 `"bool parm" [true]`

## Camera Changes
#### EnvironmentCamera is now SphericalCamera
* Declaration is now "spherical" (was "environment")
* New `string mapping` parameter with possible values of
  * equiarea (default)
  * equirect
#### PerspectiveCamera
* Remove `float halffov` parameter (just use "fov" parameter)
#### RealisticCamera
* New `float dispersionfactor` parameter (defaults to 0)
* New `float scale` parameter (defaults to 1)
* New `string aperture` parameter (defaults to "")<br>
  This parameter supports a image file path or one of the following built-ins
  * gaussian
  * square
  * pentagon
  * star

## Film Changes
#### General Film Changes
* Currently both RGBFilm and GBufferFilm have the same parameters.
* New Sensor Parameters:
  * `float exposuretime` defaults to 1
  * `float fnumber` defaults to 1
  * `float iso` defaults to 100
  * `float c` defaults to 100*PI
  * `float whitebalance` defaults to 6500
  * `string sensor` defaults to "cie1931"<br>
The sensor parameter accepts any of the following camera sensors.

| Parm Value | Common Name |
| ---- | ---- |
| cie1931 | CIE 1931 |
|canon_eos_100d | Canon EOS 100D|
|canon_eos_1dx_mkii | Canon EOS 1D X Mark II|
|canon_eos_200d | Canon EOS-200D|
|canon_eos_200d_mkii | Canon EOS 200D Mark II|
|canon_eos_5d | Canon EOS 5D|
|canon_eos_5d_mkii | Canon EOS 5D Mark II|
|canon_eos_5d_mkiii | Canon EOS 5D Mark III|
|canon_eos_5d_mkiv | Canon EOS 5D Mark IV|
|canon_eos_5ds | Canon EOS 5DS|
|canon_eos_m | Canon EOS M|
|hasselblad_l1d_20c | Hasselblad L1D-20C|
|nikon_d810 | Nikon D810|
|nikon_d850 | Nikon D850|
|sony_ilce_6400 | Sony A6400|
|sony_ilce_7m3 | Sony A7 Mark III|
|sony_ilce_7rm3 | Sony A7R Mark III|
|sony_ilce_9 | Sony A9|

#### ImageFilm is now RGBFilm
* Declaration is now "rgb" (was "image")
* `xresolution` and `yresolution` defaults have changed to (1280x720) respectively.
* `integer[4] pixelbounds` parameter added. If crop region is specificed it will override this. <br>
(this previously lived on certain integrators)
* `float maxsampleluminance` changed to `float maxcomponentvalue`
* New `bool savefp16` parameter for saving half images. (default true)
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
#### New PMJ02BN Sampler
* Declared with "pmj02bn"
* Parameters:
  * `integer pixelsamples` parameter (defaults to 16)
#### 02Sequence Sampler has become the more aptly named PaddedSobol Sampler
* Declared with "paddedsobol"
* This has the same parameters as the Sobol Sampler
#### Sobol Sampler
* New `string randomization` parameter with the following options
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
* Parameters:
  *`integer maxdepth` parameter (defaults to 5)
#### New RandomWalk Integrator
* Declared with "randomwalk"
* Parameters:
  *`integer maxdepth` parameter (defaults to 5)
#### New SimplePath Integrator
* Declared with "simplepath"
* Parameters include
  * `integer maxdepth` (defaults to 5)
  * `bool samplelights` (defaults to true)
  * `bool samplebsdf` (defaults to true)
#### New SimpleVolPath Integrator
* Declared with "simplevolapath"
* Parameters:
  * `integer maxdepth` parameter (defaults to 5)
#### Rename AO Integrator
* Type name is now "ambientocclusion", previously it was "ao"
* Remove `integer maxsamples` parameter
* New `float maxdistance` parameter (defaults to Inf)
#### BDPT Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power (default)
  * bvh
  * exhaustive *(new)*
* New `bool regularize` parameter (defaults to false)
#### MLT Integrator
* New `bool regularize` parameter (defaults to false)
#### Path Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* New `bool regularize` parameter (defaults to false)
#### SPPM Integrator
* New `integer seed` parameter (defaults to 0)<br>
(This does **not** lookup the seed value stored in `Options`)
* New `bool regularize` parameter (defaults to false)
#### VolPath Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* New `bool regularize` parameter (defaults to false)

## Light Changes
#### All Lights
* `spectrum scale` is now `float scale`
#### Goniometric Light
* `string  mapname` is now `string filename`
* New `float power` parameter defaults to -1
#### Infinite Light
* Remove `integer samples` parameter
* `string  mapname` is now `string filename`
* New `float illuminance` parameter defaults to -1
* New `point[4] portal` parameter (only supported when using `string filename`)
* `spectrum L` and `string filename` are mutually exclusive and should not be declared together.
#### Projection Light
* `string  mapname` is now `string filename`
* Remove `spectrum I` parameter<br>
* New `float power` parameter defaults to -1
*Spectrum values come only from the supplied image*
* `float fov` default has changed from 45 to 90
#### Diffuse AreaLight
* Remove `integer samples` parameter
* New "string filename" parameter (defaults to "")
* New `float power` parameter defaults to -1
* `spectrum L` and `string filename` are mutually exclusive and should not be declared together.
#### Distant Light
* New `float illuminance` parameter defaults to -1

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
* New `spectrum texture reflectance` parameter
* New `spectrum texture mfp` parameter
* New `float g` parameter
* New `float texture roughness` parameter
#### Mix Material
* Change `spectrum texture amount` to `float texture amount`

## Texture Changes
#### Remove UVTexture
#### Fbm Texture
* Output is now only float, in pbrt-v3 it was float and spectrum.
#### ImageMap Texture
* New `float scale` parmeter, (defaults to 1)
* New "octahedralsphere" mode for `string wrap`
* Remove `bool trilinear` parameter
* New `string filter` parameter, options include
  * point
  * bilinear (default)
  * trilinear
  * ewa
* Remove `bool gamma` parameter
* New `string encoding` parameter, this accepts either a builtin name or a "gamma value".<br>
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
* :exclamation:Change uv parameter from `float[] uv` to `point2[] uv`
#### Ply Shape
* Remove `texture shadowalpha` parameter

## Medium Changes
#### Homogeneous Grid
* New `spectrum Le` parameter defaults to 0
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
* Parameters:
  * General Medium parameters described above
  * `float[]|rgb[] density`
  * `integer nx` defaults to 1
  * `integer ny` defaults to 1
  * `integer nz` defaults to 1
  * `spectrum Le` defaults to 0
  * `float[] Lescale` defaults to 0
  * `point3 p0` defaults to (0,0,0)
  * `point3 p1` defaults to (1,1,1)
#### New Cloud Medium
* Declare with "cloud"
* Parameters:
  * General Medium parameters described above
  * `float density` defaults to 1
  * `float wispiness` defaults to 1
  * `float extent` defauts to 1
  * `point3 p0` defaults to (0,0,0)
  * `point3 p1` defaults to (1,1,1)  
#### New NanoVDB Medium
* Declare with "nanovdb"<br>
Looks for fields within the VDB of the name "density" and "temperature"
* Parameters:
  * General Medium parameters described above
  * `string filename` defaults to ""
  * :exclamation:`float LeScale` defaults to 1.0<br>
  Currently uniform group has a similar parameter with different case `float Lescale`
  * `float temperaturecutoff` defaults to 0.0
  * `float temperaturescale` defaults to 1.0
 
