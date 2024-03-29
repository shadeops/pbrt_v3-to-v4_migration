# Scene Description Changes from pbrt-v3 to pbrt-v4

This list is currently based off the changes made up to this commit- https://github.com/mmp/pbrt-v4/tree/cdccb7

## Table of Contents
* [Preamble](#preamble)
* [Base Scene Description](#base-scene-description-changes)
* [Camera](#camera-changes)
* [Film](#film-changes)
* [Filter](#filter-changes)
* [Sampler](#sampler-changes)
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
* :exclamation:`TransformBegin` and `TransformEnd` are deprecated, use `AttributeBegin` and `AttributeEnd` instead.
* New `Import` statement, similar to the `Include` but does not maintain graphics state (but does maintain defined objects).
* New `ColorSpace "name"` graphics state Attribute(?)<br>
Sets the ColorSpace to be "name" which defines the color space of spectrum parameters. Accepts the following builtin names -
  * srgb (default)
  * dci-p3
  * rec2020
  * aces2065-1
* New `Attribute "target" "parm" [ value ]` for setting parameter "overrides" for various targets which include -
  * shape
  * light
  * material
  * medium
  * texture
* New `Option "name" "value"` call added<br>
This sets some global options including
  * `bool disablepixeljitter` (false)
  * `bool disablewavelengthjitter` (false)
  * `bool disabletexturefiltering` (false)
  * `float displacementedgescale` (1.0)
  * `string rendercoordsys` valid values are:
    * camera
    * cameraworld (default)
    * world
  * `string msereferenceimage` ("")
  * `string msereferenceout` ("")
  * `integer seed` (0)
  * `bool forcediffuse` (false)
  * `bool pixelstats` (false)
  * `bool wavefront` (false)
* Type Changes
  * :exclamation: bool types are no longer quoted
    * pbrt-v3 `"bool parm" ["true"]`
    * pbrt-v4 `"bool parm" [true]`
  * :exclamation: spectrum type changes 
    * spectrum blackbody parameters should no longer specify an intensity value
      * pbrt-v3 `"blackbody parm" [6500 0.5]`
      * pbrt-v4 `"blackbody parm" [6500]`
    * spectrum xyz parameters are no longer supporter
      * pbrt-v3 `"xyz parm" [ 0.2 0.5 0.8 ]`
      * pbrt-v4 *not supported*

## Camera Changes
#### EnvironmentCamera is now SphericalCamera
* Declaration is now "spherical" (was "environment")
* New `string mapping` parameter with possible values of
  * equalarea (default)
  * equirectangle
#### PerspectiveCamera
* Remove `float halffov` parameter (just use "fov" parameter)
#### RealisticCamera
* New `string aperture` parameter (defaults to "")<br>
  This parameter supports a image file path or one of the following built-ins
  * gaussian
  * square
  * pentagon
  * star

## Film Changes
#### General Film Changes
* Default film is "rgb"
* Currently both RGBFilm and GBufferFilm have the same parameters.
* New Sensor Parameters:
  * `float iso` defaults to 100
  * `float whitebalance` defaults to 0<br>
*If your whitebalance is 0, and your sensor is something other than cie1931, pbrt will automatically default the sensor to 6500*
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
* Declaration is "gbuffer"
* Parameters are the same as RGBFilm with the additional -
  * `string coordinatesystem` defaults to "camera"<br>
    Possible values include:
    * "camera"
    * "world"
#### New SpectralFilm
* Declaration is "spectral"
* Parameters are the same as RGBFilm with the additional -
  * `integer buckets` defaults to 16
  * `float lambdamin` defaults to 360
  * `float lambdamax` defaults to 830
   

## Filter Changes
#### All Filters
* Rename `float xwidth` to `float xradius`
* Rename `float ywidth` to `float yradius`
#### Gaussian Filter
* `float xradius` and `float yradius` defaults are now 1.5
* `float alpha` is now `float sigma` default is 0.5

## Sampler Changes
The default sampler is "zsobol"
* All samplers have a `int seed` parameter that defaults to a global `Option "seed" value`
#### Remove MaxMinDist Sampler
#### Renamed Random Sampler to Independent Sampler
* Declared with "independent"
#### New PMJ02BN Sampler
* Declared with "pmj02bn"
* Parameters:
  * `integer pixelsamples` parameter (defaults to 16)
#### New ZSobolSampler
* Declared with "zsobol"
* This has the same parameters as the Sobol Sampler
#### 02Sequence Sampler has become the more aptly named PaddedSobol Sampler
* Declared with "paddedsobol"
* This has the same parameters as the Sobol Sampler
#### Sobol Sampler
* New `string randomization` parameter with the following options
  * none
  * owen
  * fastowen (default)
  * permutedigits
#### Stratified Sampler
* Remove `integer dimensions` parameter
#### Halton Sampler
* Remove `bool samplepixelcenter` parameter
* New `string randomization` parameter with the following options
  * none
  * owen
  * permutedigits (default)

## Integrator Changes
#### General
* `integer[4] pixelbounds` that existed on the various Integrators has been removed and now live on Film
* Default integrator is now  VolumePath
#### Remove Whitted Integrator
#### Remove DirectLighting Integrator
#### New LightPath Integrator
* Declared with "lightpath"
* Parameters:
  * `integer maxdepth` defaults to 5
#### New RandomWalk Integrator
* Declared with "randomwalk"
* Parameters:
  * `integer maxdepth` defaults to 5
#### New Function Integrator
* Declared with "function"
* Parameters:
  * `string function`
  The following options are supported
    * step (default)
    * diagonal
    * disk
    * checkerboard
    * rotatedcheckerboard
    * gaussian
  * `bool skipbad` (true)
  * `string filename` defaults to Options->imageFile if supplied otherwise {function}-mse.txt
  * `string imagefilename` ("")
#### New SimplePath Integrator
* Declared with "simplepath"
* Parameters:
  * `integer maxdepth` defaults to 5
  * `bool samplelights` defaults to true
  * `bool samplebsdf` defaults to true
#### New SimpleVolPath Integrator
* Declared with "simplevolapath"
* Parameters:
  * `integer maxdepth` defaults to 5
#### Rename AO Integrator
* Type name is now "ambientocclusion", previously it was "ao"
* Remove `integer maxsamples` parameter
* New `float maxdistance` parameter, defaults to Inf
#### BDPT Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * :exclamation: power (default)<br>
  *BDPT's lightsampler default is different from Path/VolPath*
  * bvh
  * exhaustive *(new)*
* New `bool regularize` parameter, defaults to false
#### MLT Integrator
* New `bool regularize` parameter, defaults to false
#### Path Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* New `bool regularize` parameter, defaults to false
* Remove `float rrthreshold` parameter
#### SPPM Integrator
* New `integer seed` parameter, defaults to 6502<br>
(This does **not** lookup the seed value stored in `Options`)
* Remove `integer iterations` parameter
#### VolPath Integrator
* Replace `string lightsamplestrategy` with `string lightsampler`<br>
The following options are supported
  * uniform
  * power
  * bvh (default)
  * exhaustive *(new)*
* New `bool regularize` parameter, defaults to false
* Remove `float rrthreshold` parameter

## Light Changes
#### All Lights
* `spectrum scale` is now `float scale`
* New `float power` parameter defaults to -1
#### Goniometric Light
* `string  mapname` is now `string filename`
#### Infinite Light
* Remove `integer samples` parameter
* `string  mapname` is now `string filename`
* New `float illuminance` parameter, defaults to -1
* New `point[4] portal` parameter
* `spectrum L` and `string filename` are mutually exclusive and should not be declared together.
   If `portal` and `L` are used, `L` is converted to an RGB texture.
#### Projection Light
* `string  mapname` is now `string filename`
* Remove `spectrum I` parameter<br>
*Spectrum values come only from the supplied image*
* New `float power` parameter, defaults to -1
* `float fov` default has changed from 45 to 90
#### Diffuse AreaLight
* Remove `integer samples` parameter
* New "string filename" parameter, defaults to ""
* New `float power` parameter, defaults to -1
* `spectrum L` and `string filename` are mutually exclusive and should not be declared together.
* Alpha textures act on emission similar to visiblity. A special case when alpha is a constant 0 will result in an emissive invisible light source.
#### Distant Light
* New `float illuminance` parameter, defaults to -1

## Material Changes
#### General Changes
* `float texture bumpmap` has been renamed to `float texture displacement`
* `string normalmap` has been added, defaults to ""
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
#### None Material renamed Interface Material
* Declared with "interface", lights with an interface material are ignored.
#### CoatedConductor Material
Type name: "coatedconductor"
##### Parameters:
* `float texture displacement` defaults to null
* `spectrum interface.eta` defaults to 1.5
* `float texture thickness` defaults to 0.01
* `float texture interface.roughness` defaults to 0
* `float texture interface.uroughness` defaults to `interface.roughness`
* `float texture interface.vroughness` defaults to `interface.roughness`
* `spectrum texture conductor.eta` defaults to "metal-Cu-eta"
* `spectrum texture conductor.k` defaults to "metal-Cu-k"
* `spectrum texture reflectance` defaults to null, not to be used with `conductor.eta` and `conductor.k`
* `float texture conductor.roughness` defaults to 0
* `float texture conductor.uroughness` defaults to `conductor.roughness`
* `float texture conductor.vroughness` defaults to `conductor.roughness`
* `bool remaproughness` defaults to true
* `integer maxdepth` defaults to 10
* `integer nsamples` defaults to 1
* `float texture g` defaults to 0
* `spectrum texture albedo` defaults to 0
#### CoatedDiffuse Material
Type name: "coateddiffuse"
##### Parameters:
* `float texture displacement` defaultsto null
* `spectrum texture reflectance` defaults to 0.5
* `spectrum eta` default 1.5
* `float texture thickness` default 0.01
* `float texture roughness` defaults to 0
* `float texture uroughness` defaults to `roughness`
* `float texture vroughness` defaults to `roughness`
* `bool remaproughness` defaults to true
* `integer maxdepth` defaults to 10
* `integer nsamples` defaults to 1
* `float texture g` defaults to 0
* `spectrum texture albedo` defaults to 0
#### Conductor Material
Type name: "conductor"
##### Parameters:
* `float texture displacement` defaults to null
* `spectrum texture eta` defaults to "metal-Cu-eta"
* `spectrum texture k` defaults to "metal-Cu-k"
* `spectrum texture reflectance` defaults to null, not to be used with `eta` and `k`
* `float texture roughness` defaults to 0
* `float texture uroughness` defaults to `roughness`
* `float texture vroughness` defaults to `roughness`
* `bool remaproughness` defaults to true
#### Diffuse Material
Type name: "diffuse"
##### Parameters:
* `float texture displacement` defaults to null
* `spectrum texture reflectance` defaults to 0.5
#### DiffuseTransmission Material
Type name: "diffusetransmission"
##### Parameters:
* `float texture displacement` defaults to null
* `spectrum texture reflectance` defaults to 0.25
* `spectrum texture transmittance` defaults to 0.25
* `float scale` defaults to 1
#### Measured Material
Type name: "measured"
##### Parameters:
* `float texture displacement` defaults to null
* `string filename` defaults to ""
#### ThinDielectric Material
Type name: "thindielectric"
##### Parameters:
* `float texture displacement` defaults to null
* `spectrum eta` defaults to 1.5
#### Dielectric Material
Type name: "dielectric"
##### Parameters:
* `float texture displacement` defaults to null
* `spectrum eta` defaults to 1.5
* `float texture roughness` defaults to 0
* `float texture uroughness` defaults to `roughness`
* `float texture vroughness` defaults to `roughness`
* `bool remaproughness` defaults to true
#### Subsurface Material
* Remove `spectrum Kr` parameter
* Remove `spectrum Kt` parameter
* New `spectrum texture reflectance` defaults to 1
* New `spectrum texture mfp` defaults to 1
* New `float g` defaults to 0
* New `float texture roughness` defaults to 0
#### Mix Material
* Change `spectrum texture amount` to `float texture amount`
* Remove `string namedmaterial1` and `string namedmaterial2`
* Add `string[2] materials`
#### Hair Material
* Change `spectrum texture color` to `spectrum texture reflectance` (color still works if reflectance isn't specified)

## Texture Changes
#### Remove UVTexture
#### New DirectionMix Texture
* Supports both float and spectrum
* Parameters are
  * `float|spectrum texture tex1` defaults to 0.0
  * `float|specturm texture tex2` defaults to 1.0
  * `vector3 dir` defaults to (0.0, 1.0, 0.0)
#### Fbm Texture
* Output is now only float, in pbrt-v3 it was float and spectrum.
#### ImageMap Texture
* New `float scale` parmeter, defaults to 1
* New "octahedralsphere" mode for `string wrap`
* Remove `bool trilinear` parameter
* New `string filter` parameter, options include
  * point
  * bilinear (default)
  * trilinear
  * ewa
* Remove `bool gamma` parameter
* New `bool invert` parameter, defaults to false
* New `string encoding` parameter, this accepts either a builtin name or a "gamma value".<br>
One of the following options can be specified. (The default is based on the file extension of the texture.)
  * linear
  * sRGB
  * gamma float_value
#### Marble Texture
* Output is now only spectrum, in pbrt-v3 it was float and spectrum
#### Scale Texture
* `texture tex1` is now just `texture tex`
* `texture tex2` has been replaced with `float scale`, defaults to 1
#### Windy Texture
* Output is now only float, in pbrt-v3 it was float and spectrum.
#### Wrinkled Texture
* Output is now only float, in pbrt-v3 it was float and spectrum.
#### Ptex Texture
* New `float scale` parameter, defaults to 1

## Shape Changes
#### General
* All shapes now support an `float texture alpha` parameter.
* :exclamation: Shapes no longer support overriding of a Material's parameters as described here https://www.pbrt.org/fileformat-v3.html#materials

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
* Add `float texture displacement` parameter, defaults to null (deactivated)
* Add `float edgelength` parameter, defaults to 1.0

## Medium Changes
#### Homogeneous Medium
* New `spectrum Le` parameter, defaults to 0
* `string preset` default ("") Only exists on Homogeneous Medium
* The list of presets scattering properties has not changed since pbrt-v3*
#### New Uniform Medium
This was previously the Heterogeneous Medium in pbrt-v3
* Declare with "uniformgrid"
* Parameters:
  * General Medium parameters described above
  * `float[] density`
  * `float[] temperature`
  * `float[] Lescale`
  * Only one combination of Le or temperature is allowed
  * `spectrum sigma_a` defaults to (1, 1, 1)
  * `spectrum sigma_s` defaults to (1, 1, 1)
  * `spectrum Le` defaults to 0
  * `float scale` defaults to 1.0
  * `float g` defaults to 0.0
  * `integer nx` defaults to 1
  * `integer ny` defaults to 1
  * `integer nz` defaults to 1
  * `point3 p0` defaults to (0,0,0)
  * `point3 p1` defaults to (1,1,1)
#### New RGBGrid Medium
* Declare with "rgbgrid"
* Parameters:
  * `rgb[] sigma_a`
  * `rgb[] sigma_s`
  * `rgb[] Le`
  * Can specify sigma_a and/or sigma_s. But if Le is used then sigma_a is required.
  * `float LeScale` defaults to 0
  * `float scale` defaults to 1.0
  * `float g` defaults to 0.0
  * `integer nx` defaults to 1
  * `integer ny` defaults to 1
  * `integer nz` defaults to 1
  * `point3 p0` defaults to (0,0,0)
  * `point3 p1` defaults to (1,1,1)
#### New Cloud Medium
* Declare with "cloud"
* Parameters:
  * `float density` defaults to 1
  * `float wispiness` defaults to 1
  * `float frequency` defauts to 5
  * `spectrum sigma_a` defaults to (1, 1, 1)
  * `spectrum sigma_s` defaults to (1, 1, 1)
  * `spectrum Le` defaults to 0
  * `float g` defaults to 0.0
  * `point3 p0` defaults to (0,0,0)
  * `point3 p1` defaults to (1,1,1)  
#### New NanoVDB Medium
* Declare with "nanovdb"<br>
Looks for fields within the VDB of the name "density" and "temperature"
* Parameters:
  * General Medium parameters described above
  * `string filename` defaults to ""
  * `float LeScale` defaults to 1.0
  * `float temperaturecutoff` defaults to 0.0
  * `float temperaturescale` defaults to 1.0
  * `spectrum sigma_a` defaults to (1, 1, 1)
  * `spectrum sigma_s` defaults to (1, 1, 1)
  * `float scale` defaults to 1.0

 
