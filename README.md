# Scene Description Changes from pbrt-v3 to pbrt-v4

## Preamble
This is not meant to be the official document representing the changes between pbrt-v3 and pbrt-v4, I'll leave that to Matt's much more capable hands. These are my notes during the process of updating an exporter from v3 to v4. I am making these public to aid others in the transition until a formal page is created by Matt. The changes listed are deduced from reading through the source code of https://github.com/mmp/pbrt-v4. As pbrt-v4 is still an early release and changing rapidly expect these notes to be out of sync at times. When in doubt, trust the code! :)

### Updating an exporter while the interface is still undergoing changes?
Yup, I'm an idiot. But its exciting to follow the developments and try out the changes!

## Base Scene Description Changes
* `WorldEnd` has been removed
* bool types are no longer quoted
  * pbrt-v3 `"bool parm" ["true"]`
  * pbrt-v4 `"bool parm" [true]`


## Camera Updates
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
