# HOW TO Build & Run MXNet-cpp

## Pre1: Build MXNet from source

See [Build MXNet from source](Build_MXNet_from_source.md) for more detail. 

## Build MXNet-cpp

We need to add the MXNet static library `libmxnet.a` to `LIBRARY_PATH` and the dynamic library `libmxnet.so` to `LD_LIBRARY_PATH`.

I've copied those libraries under the `mxnet-cpp/example directory`, and add current directory `.` to `LIBRARY_PATH` and `LD_LIBRARY_PATH`.

Until the v0.10.0 release, building mxnet-cpp will generate a lack of `mxnet-cpp/op.h`. Just `touch` a new one.

## Updated at 2017-07-18

I decided to abandon MXNet-cpp, since I didn't discover any kvstore usage in MXNet-cpp.