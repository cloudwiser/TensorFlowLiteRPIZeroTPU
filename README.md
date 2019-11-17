<img  align="left"  width="120"  height="120"  src="https://avatars0.githubusercontent.com/u/15658638?s=200&v=4">
<img  align="centre"  width="85"  height="30"  src="https://codelabs.developers.google.com/codelabs/edgetpu-classifier/img/9f1dccf093ab1f6d.png">

### TensorFlow Lite & Coral TPU: C++ examples on Raspberry Pi Zero W

This guide outlines the steps to get the `minimal` C++ example provided in the Google Coral TPU `edgetpu` distro running on the Raspberry Pi Zero W

If you want to just get up-and-running then Google provide an SD image for the Raspberry Pi Zero W at https://github.com/google-coral/edgetpu-platforms/releases/tag/v1.9.2 ...but if you want to understand the build process and potential issues you might hit, read on!

First of all, you'll need to purchase a Coral TPU USB stick from https://coral.withgoogle.com/products/

You will also need a USB-to-micro-USB adapter if you intend to use the cable provided with Coral TPU adapter to connect to the RPi Zero USB port (the one nearest the SD card). I decided to skip the adapter and purchase a suitable USB-C-to-micro-USB e.g. https://smile.amazon.co.uk/gp/product/B01LONQ7R6

I haven't checked the power consumptiom of the Coral TPU plus RPi Zero W but please check that your micro-USB power source can drive both devices

Once you plug-in the TPU adapter, the white LED should illuminate. You should now check that it is recognised as a USB device via...

```sh
pi@raspberrypi: <path> $ lsusb
Bus 001 Device 003: ID 18d1:9302 Google Inc.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Note that if the (Google) device doesn't appear, check you are not running other USB drivers e.g. a virtual network connection from your client to your RPi Zero W via USB. If you are, remove your SD and edit the image to amend cmdline.txt, config.txt, etc. files to disable these drivers

Next you need to build the `lib_tensorflow-lite.a` in `<path_to>/tensorflow/tensorflow/lite/tools/make/gen/rpi_armv6/lib`. I have a repo that outlines this process (warning: it takes time if you are native compiling on the RPi Zero W) at https://github.com/cloudwiser/TensorFlowLiteRPIZero

With this complete and working on the non-TPU samples, proceed with the `edgetpu` runtime install as per step #1 at https://coral.withgoogle.com/docs/accelerator/get-started/#1-install-the-edge-tpu-runtime

Also, ensure you have the the required external dependencies via...
```sh
$ <path_to>/tensorflow/tensorflow/lite/tools/make/download_dependencies.sh`
```

FWIW, I installed the runtime package under `~/tensorflow/edgetpu`...so if you hit build problems, it maybe down to path dependencies

Note I didn't proceed any further through the Getting Started section as this is focussed on running the python samples and (at the time of writing) there was no python wheel for armv6 at https://www.tensorflow.org/lite/guide/python

Now for a tweak to `/usr/lib`...as the `edgetpu` runtime places `libedgetpu.so.1.0` and a link to it in `/usr/lib/arm-linux-gnueabihf`...unfortunately for the RPi Zero W, this is a version for armv7. If we don't amend this, applications will compile and link and then core dump with `Illegal instruction` when run!

We need to overwrite this lib with the armv6 version so...copy the armv6 library from `libedgetpu.so.1.0` from `<path_to>/edgetpu/libedgetpu/direct/armv6` to `/usr/lib/arm-linux-gnueabihf`

This may need you to `sudo cp <src> <dest>` plus you should also to change the file permissions and group:owner via `chmod` and `chown` respectively

Now rename the original `Makefile` in `<path_to>/tensorflow/edgetpu/src/cpp/examples` to ``Makefile.orig` and replace it the `Makefile` from this repo

Edit the new `Makefile` and set your `TENSORFLOW_DIR` to the relevant path

Run `make` to compile and link `minimal.cc` and `model_utils.cc`
Check you have the `minimal` binary if the make completes silently

Next we require the tflite model and source image for `minimal` so copy the `mobilenet_v1_1.0_224_quant_edgetpu.tflite` and `resized_cat.bmp` files from `<path_to>/tensorflow/edgetpu/test_data` into this `examples` directory

Ensure your Google Coral TPU USB stick is installed and run...
```sh
pi@raspberrypi: <path> $ ./minimal mobilenet_v1_1.0_224_quant_edgetpu.tflite resized_cat.bmp
```

The output should be similar to/same as...
```sh
pi@raspberrypi: <path> $ ./minimal mobilenet_v1_1.0_224_quant_edgetpu.tflite resized_cat.bmp
[Image analysis] max value index: 286 value: 0.792969
```

Happy TensorFlow-Lite-development-with-TPU-inference-on-arm6-devices!
