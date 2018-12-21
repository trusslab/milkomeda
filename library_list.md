Copyright (c) 2018 University of California, Irvine. All rights reserved.

See our [Milkomeda paper](https://dl.acm.org/citation.cfm?id=3243772) for technical details.
Authors: Zhihao Yao, Saeed Mirzamohammadi, Ardalan Amiri Sani, Mathias Payer

This document is shared under the GNU Free Documentation License WITHOUT ANY WARRANTY. See <https://www.gnu.org/licenses/> for details.

### List of modified system libraries

`->` means a rename.

```
/data/local/lib64/libt.so
/system/lib65/libnetd_client.so -> litnetd_client.so (sed libt)
/system/lib65/libc++.so -> litc++.so (sed libt)
/system/lib65/libm.so -> litm.so (sed libt)

/vendor/lib65/libgsl.so (sed libt, libcutils, litlog, libz, libc++)
/system/lib65/libcutils.so -> libcutils.so (sed libt, libm, libc++)
/system/lib65/liblog.so -> litlog.so (sed libt, libm)
/system/lib65/libz.so -> litz.so (sed libt, libm)

/vendor/lib65/libadreno_utils.so (sed libt, libc++, libm)

/vendor/lib65/libllvm-glnext.so (sed libt, libc++)

/vendor/lib65/egl/libEGL_adreno.so (sed libt, libcutils, liblog, libz, libc++, libm)

/vendor/lib65/egl/libGLESv1_CM_adreno.so (sed libt, libc++)

/vendor/lib65/egl/eglSubDriverAndroid.so (sed libt, libhardware, libcutils, libc++)
/system/lib65/libhardware.so (sed libt, liblog, libc++, libm)

/system/lib65/hw/gralloc.msm8992.so -> tralloc.msm8992.so (libt, libc++, libm, libcutils, liblog, libhardware, libutils, libmemalloc, libqdMetaData, libqdutils)
/system/lib65/libutils.so -> litutils.so (sed libt, libcutils, liblog, libc++, libm)
/system/lib65/libmemalloc.so -> litmemalloc.so (sed libt, libcutils, liblog, libc++, libm, libhardware, libutils)
/system/lib65/libqdMetaData.so -> litqdMetaData.so (sed libt, libcutils, liblog, libc++, libm)
/system/lib65/libqdutils.so -> litqdutils.so (sed libt, libcutils, liblog, libc++, libm, libhardware, libutils)

/system/lib65/libgui.so ->	litgui.so (sed libt, libcutils, liblog, libc++, libm, libutils, libui, libbinder, libsync, libnativeloader)
/system/lib65/libnativeloader.so -> litnativeloader.so (sed libt, liblog, libc++, libm, libcutils)
/system/lib65/libui.so -> litui.so (sed libt, libcutils, liblog, libc++, libm, libhardware, libutils)
/system/lib65/libbinder.so  -> litbinder.so(sed libt, libcutils, liblog, libc++, libm, libutils)
/system/lib65/libsync.so  -> litsync.so(sed libt, liblog, libc++, libm)
```