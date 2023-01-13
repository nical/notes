
Source code: https://github.com/google/skia

* Seems to be using a mix of:
  * tessellation
    * Half-edge mesh internal representation for paths
  * stencil-and-cover
  * software fallback with texture upload

TODO: I need to check this, but the code at least contains all of the above.

* Skia tessellator presentation: https://docs.google.com/presentation/d/1tyroXtcGwOvU1LPFxVU-vtBiDkLTcxZ62v2S9wqZ77w/edit#slide=id.gb26fbbc26_0_128
* SKia vertex aa presentation: https://docs.google.com/presentation/d/1DpM5QS6kCkIqQN034Zz6oFm201Gd2wvq6Z30QfWNhcA/edit#slide=id.g18663b9d41_2_0
