# Documentation for setting up running environment for Tensorflow C++ API without bazel


## Possible problems:

 * Libtensorflow_framework.so not found! 

Locate file at `~/Desktop/tensorflow/bazel-bin/tensorflow`. Need to run `sudo ldconfig` to refresh linker.
 
 * Nsync.h not found!

Download at [nsync git repos](https://github.com/google/nsync/tree/master/public), and place it in the right folder `tensorflow/core/platform/default/`.

* Tensorflow version = `1.3.0`; Bazel version = `0.6.1`; Protocbuff version = `3.4.0`


* Remember to run `./configure` in tensorflow main repo before run bazel command `bazel build //tensorflow:libtensorflow_cc.so`.


* `*.pb.h` missing.

[Follow this discussion.](https://github.com/tensorflow/tensorflow/issues/1890)

## Steps:

1. Install `protocbuf` and `eigen`.

2. 

```sh
./configure
bazel build  //tensorflow:libtensorflow_cc.so
```

3. Then Copy the following include headers and dynamic shared library to `/usr/local/lib` and `/usr/local/include`:
```sh
mkdir /usr/local/include/tf
cp -r bazel-genfiles/ /usr/local/include/tf/
cp -r tensorflow /usr/local/include/tf/
cp -r third_party /usr/local/include/tf/
cp -r bazel-bin/libtensorflow_cc.so /usr/local/lib/
```

4. Lastly, compile using an example:
```sh
g++ -std=c++11 -o tTest test.cc -I/usr/local/include/tf -I/usr/local/include/eigen3 -g -Wall -D_DEBUG -Wshadow -Wno-sign-compare -w -L/usr/local/lib/libtensorflow_cc -ltensorflow_cc -L/usr/local/lib/libtensorflow_framework -ltensorflow_framework `pkg-config --cflags --libs protobuf`
```

### Related discussions:
* [tensorflow mobile : pi_examples - unable to locate graph.pb.h #3251](https://github.com/tensorflow/tensorflow/issues/3251)

* [github: graph.pb.h missing #1890](https://github.com/tensorflow/tensorflow/issues/1890)

* [See comments about undefined reference.](http://tuatini.me/building-tensorflow-as-a-standalone-project/)

* [Missing nsync](https://github.com/tensorflow/tensorflow/issues/12482)
