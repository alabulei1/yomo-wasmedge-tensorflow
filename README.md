# Streaming Image Recognition by WebAssembly

[![Youtube: YoMo x WasmEdge](youtube.png)](https://youtu.be/E0ltsn6cLIU)

This project demonstrates how to process a video stream in real-time using WebAssembly and apply a pre-trained [food classification model](https://tfhub.dev/google/lite-model/aiy/vision/classifier/food_V1/1) to each frame of the video in order to determine if food is present in that frame, all by integrating [WasmEdge](https://github.com/WasmEdge/WasmEdge) into [YoMo](https://github.com/yomorun/yomo) serverless.

Open-source projects that we used:

- Serverless stream processing framework [YoMo](https://github.com/yomorun/yomo)
- Integrate with [WasmEdge](https://github.com/WasmEdge/WasmEdge) to introduce WebAssembly, interop TensorFlow Lite model
- A deep learning model found on [TensorFlow Hub](https://tfhub.dev/google/lite-model/aiy/vision/classifier/food_V1/1); make sure to download `TFLite (aiy/vision/classifier/food_V1)`, which was created by Google

**Advantages:**

- ⚡️ **Low-latency**: Streaming data processing applications can now be done in far edge data centers thanks to YoMo's highly efficient network services
- 🔐 **Security**: WasmEdge runs the data processing function in a WebAssembly sandbox for isolation, safety, and hot deployment
- 🚀 **High Performance**: Compared with popular containers, such as Docker, WasmEdge can be up to 100x faster at startup and have a much smaller footprint
- 🎯 **Edge Devices**: As WasmEdge consumes much less resources than Docker, it is now possible to run data processing applications on edge devices

## Steps to run

### 1. Clone This Repository

```bash
$ git clone https://github.com/yomorun/yomo-wasmedge-tensorflow.git
```

### 2. Install YoMo CLI

```bash
$ go install github.com/yomorun/cli/yomo@latest
$ yomo version
YoMo CLI version: v0.0.5
```

Or, you can download the pre-built binary tarball [yomo-v0.0.5-x86_64-linux.tgz](https://github.com/yomorun/cli/releases/tag/v0.0.5).

Details about `YoMo CLI` installation can be found [here](https://github.com/yomorun/yomo).

### 3. Install WasmEdge Dependencies

#### Install WasmEdge

```bash
$ wget https://github.com/WasmEdge/WasmEdge/releases/download/0.8.0/WasmEdge-0.8.0-manylinux2014_x86_64.tar.gz
$ tar -xzf WasmEdge-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo cp WasmEdge-0.8.0-Linux/include/wasmedge.h /usr/local/include
$ sudo cp WasmEdge-0.8.0-Linux/lib64/libwasmedge_c.so /usr/local/lib
$ sudo ldconfig
```

Or, you can [build from the source](https://github.com/second-state/WasmEdge-go#option-1-build-from-the-source).

#### Install WasmEdge-tensorflow dependencies


```bash
$ wget https://github.com/second-state/WasmEdge-go/releases/download/v0.8.1/install_wasmedge_tensorflow_deps.sh
$ wget https://github.com/second-state/WasmEdge-go/releases/download/v0.8.1/install_wasmedge_tensorflow.sh
$ sudo ./install_wasmedge_tensorflow_deps.sh /usr/local
$ sudo ./install_wasmedge_tensorflow.sh /usr/local
```


#### Install WasmEdge-image Dependencies:

When linking with `Wasmedge-image`, `libjpeg.so.8` and `libpng16.so.16` are required.

For `ubuntu 18.04` or later, the following commands can install these dependencies:
```bash
$ sudo apt-get update
$ sudo apt-get install -y libjpeg-dev libpng-dev
```

Or you can download and install the pre-built shared libraries for the `manylinux1` platforms:

```bash
$ wget https://github.com/second-state/WasmEdge-go/releases/download/v0.8.1/install_wasmedge_image_deps.sh
$ sudo ./install_wasmedge_image_deps.sh /usr/local
```

Finally, install the `WasmEdge-image`:

```bash
$ wget https://github.com/second-state/WasmEdge-go/releases/download/v0.8.1/install_wasmedge_image.sh
$ sudo ./install_wasmedge_image.sh /usr/local
```

If you have any questions about installation, please refer to [the official documentation](https://github.com/second-state/WasmEdge-go#wasmedge-tensorflow-shared-library-installation). Currently, this project works on Linux machines only.



### 4. Write your Streaming Serverless function

Write [app.go](https://github.com/yomorun/yomo-wasmedge-tensorflow/blob/main/flow/app.go) to integrate `WasmEdge-tensorflow`:

Get `WasmEdge-go`:

```bash
$ cd flow
$ go get -u github.com/second-state/WasmEdge-go/wasmedge
```

Download pre-trained TensorflowLitee model: [lite-model_aiy_vision_classifier_food_V1_1.tflite](https://storage.googleapis.com/tfhub-lite-models/google/lite-model/aiy/vision/classifier/food_V1/1.tflite), store to `rust_mobilenet_foods/src`:

```bash
$ wget 'https://storage.googleapis.com/tfhub-lite-models/google/lite-model/aiy/vision/classifier/food_V1/1.tflite' -O ./rust_mobilenet_food/src/lite-model_aiy_vision_classifier_food_V1_1.tflite
```
Compile to `wasm` file:

Install [rustc and cargo](https://www.rust-lang.org/tools/install)

```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ export PATH=$PATH:$HOME/.cargo/bin
$ rustc --version
```

Set default `rust` version to `1.50.0`: `$ rustup default 1.50.0`

Install [rustwasmc](https://github.com/second-state/rustwasmc)

```bash
$ curl https://raw.githubusercontent.com/second-state/rustwasmc/master/installer/init.sh -sSf | sh
$ cd rust_mobilenet_food
$ rustwasmc build
# The output WASM will be `pkg/rust_mobilenet_food_lib_bg.wasm`.
```

Copy `pkg/rust_mobilenet_food_lib_bg.wasm` to `flow` directory:

```bash
$ cp pkg/rust_mobilenet_food_lib_bg.wasm ../.
```

### 5. Run YoMo Orchestrator Server

```bash
  $ yomo serve -c ./zipper/workflow.yaml
```

### 6. Run Streaming Serverless function

```bash
$ cd flow
$ go run --tags "tensorflow image" app.go
```

### 7. Demonstrate video stream

Download [this demo vide: hot-dog.mp4](https://github.com/yomorun/yomo-wasmedge-tensorflow/releases/download/v0.1.0/hot-dog.mp4), store to `source` directory, then run：

```bash
$ wget -P source 'https://github.com/yomorun/yomo-wasmedge-tensorflow/releases/download/v0.1.0/hot-dog.mp4'
$ go run ./source/main.go ./source/hot-dog.mp4
```

### 8. Result

![YoMo-WasmEdge](result.png)
