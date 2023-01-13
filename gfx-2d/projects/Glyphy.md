
Source code: https://github.com/behdad/glyphy

* Vector texture
  * Approximates all curves with elliptic arc segments
  * Very little cpu work except for bézier -> arc approximation
  * Sensitive to driver bugs
  * Gotta trust the driver not to fiddle with texture formats behind our back, otherwise the path representation is corrupted.
  * No overdraw mitigation

