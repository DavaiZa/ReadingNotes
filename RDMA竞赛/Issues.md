# MXNet如何整合原生InfiniBand?

**Origin:** [How to use native Infiniband instead of IPoIB](https://github.com/dmlc/mxnet/issues/4766)

**Conclusion:** MXNet didn't support it. be more accurate, **ps-lite doesn't support native IB**. If you want to contribute, please take a look project dmlc/ps-lite. a good start point will be class Van (reference include/ps/internal/van.h) You can leverage RDMA to do message send/receive.

**Notes:**

* ps-lite是作为submodule整合进mxnet的