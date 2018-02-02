
# Documentation for setting up running environment for Tensorflow C++ API without bazel
[Updated 2/2/2018]


## Possible problems:

 * Libtensorflow_framework.so not found! 

    Locate file at `~/Desktop/tensorflow/bazel-bin/tensorflow`. Need to run `sudo ldconfig` to refresh linker.
 
 * Nsync.h not found!

    Download at [nsync git repos](https://github.com/google/nsync/tree/master/public), and place it in the right folder `tensorflow/core/platform/default/`.

* Tensorflow version = `1.5.0-rc1/master`; Bazel version = `0.9.0`; Protocbuff version = `3.4.0`


* Remember to run `./configure` in tensorflow main repo before run bazel command `bazel build //tensorflow:libtensorflow_cc.so`.


* `*.pb.h` missing.

[Follow this discussion.](https://github.com/tensorflow/tensorflow/issues/1890)

* `libcupti.*` missing or no `extras/CUPTI` related problem
1. Install `libcupti8.0` and `libcupti-dev` from deb packages. See [this](https://github.com/tensorflow/tensorflow/issues/9341#issuecomment-324041125) post.
2. create etras/CUPTI directory and link it with libcupti libraries. See [this](https://github.com/tensorflow/tensorflow/issues/3526#issuecomment-235882334) post.



## Steps:

1. Install `protocbuf`, `bazel` and `eigen`.

    `eigen` comes with VENTOS already, so no need to install it again. 
    
    protobuf need to be installed from source code. Follow the instruction in [here](https://github.com/hjchai/tensorflow-c-api/blob/master/install%20protobuf) to install protobuf from source. 

    `bazel` released version can be downloaded from [here](https://github.com/bazelbuild/bazel/releases). Use the following command to install `bazel`:

    ```sh
    sudo dpkg -i bazel_0.9.0-linux-x86_64.deb
    ```
    
    As for tensorflow version v1.5.0-rc1 and bazel 0.9.0, only protobuf v3.4.0 works. (As Feb 2, 2018, bazel 0.10.0 does not work with tensorflow v1.5.0-rc1. Use bazel 0.9.0 instead.).
2. Install `cuda-8.0` and `cudnn-6.1`. Follow [this](https://github.com/hjchai/tensorflow-c-api/blob/master/install%20cuda%20and%20cudnn) instruction.
2. Start to build tensorflow libraries.

```sh
./configure
bazel build  //tensorflow:libtensorflow_cc.so
```

3. Merge `bazel-genfiles/tensorflow` with `tensorlow/tensorflow`

4. Then Copy the following include headers and dynamic shared library to `/usr/local/lib` and `/usr/local/include` (You are at `tensorflow` main folder):
```sh
cp -r tensorflow /usr/local/include/
cp -r third_party /usr/local/include/
cp -r bazel-bin/tensorflow/libtensorflow_cc.so /usr/local/lib/
cp -r bazel-bin/tensorflow/libtensorflow_framework.so /usr/local/lib/
```

4. Lastly, compile using an example:
```sh
g++ -std=c++11 -o tTest test.cc -I/usr/local/include/tf -I/usr/local/include/eigen3 -g -Wall -D_DEBUG -Wshadow -Wno-sign-compare -w -L/usr/local/lib/libtensorflow_cc -ltensorflow_cc -L/usr/local/lib/libtensorflow_framework -ltensorflow_framework `pkg-config --cflags --libs protobuf`
```

## How to port to VENTOS

1. Add `-I/usr/local/include/eigen3` to `Property->OMNeT++->Makemake->src->Options->Preview`.

2. Add `tensorflow_cc` and `tensorflow_framework` to `Property->OMNeT++->Makemake->src->Options->Link`.

3. Include `tensorflow` head files.

## Graph building
* [This talks about how to load checkpoint to C++](https://stackoverflow.com/questions/35508866/tensorflow-different-ways-to-export-and-run-graph-in-c/43639305#43639305)

* [In this post one can find how to save to checkpoint in C++](https://github.com/tensorflow/tensorflow/issues/11236)

* [In this post one can find an example how to load a graph into C++](https://tebesu.github.io/posts/Training-a-TensorFlow-graph-in-C++-API)

* [In this post one can find a detailed example on how to save/load a checkpoint in Python](http://cv-tricks.com/tensorflow-tutorial/save-restore-tensorflow-models-quick-complete-tutorial/)

Note on exporting graph and model in tensorflow:

1. Simple tensorflow graph file *.pb only contains graph topology information. It doesn't contain any variables like weight or bias information.

2. Metagraph(checkpoint files) contains graph information in *.meta, and variable values in *.data and *.index files.

3. One can consolidate \*.pb and \*.meta/\*.data/\*.index into a single \*.pb file which is known as frozen graph. (Python API provides a frozen_graph.py script to freeze a graph)

4. However, frozen graph can only be used for inference. One can reload the frozen graph in other applications and use it to do tasks like prediction. But it can't be used for further training. This is because the variables are converted to constants when the graph is frozen.

5. In order to make the saved graph applicable for continuing training, the following approach can be used (at least for TF version 1.3.0): load checkpoint files. This is so far the only possible way to reuse a pre-trained model for continuing training purpose.

### Related discussions:
* [tensorflow mobile : pi_examples - unable to locate graph.pb.h #3251](https://github.com/tensorflow/tensorflow/issues/3251)

* [github: graph.pb.h missing #1890](https://github.com/tensorflow/tensorflow/issues/1890)

* [See comments about undefined reference.](http://tuatini.me/building-tensorflow-as-a-standalone-project/)

* [Missing nsync](https://github.com/tensorflow/tensorflow/issues/12482)

> Written with [StackEdit](https://stackedit.io/).
