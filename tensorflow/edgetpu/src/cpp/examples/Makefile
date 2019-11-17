# ----------------------------------------------------------------------
#   Makefile to build the minimal.cc example on armv6 RPi Zero W
#
#   See https://github.com/cloudwiser/TensorFlowLiteRPIZeroTPU for the
#   complete set of preequisite steps
#
#   Nick Hall  https://github.com/cloudwiser/   http://ai.cloudwise.co  
# ----------------------------------------------------------------------

MAKEFILE_DIR := $(realpath $(dir $(lastword $(MAKEFILE_LIST))))
TENSORFLOW_DIR := /home/pi/tensorflow

minimal: minimal.cc model_utils.cc
	arm-linux-gnueabihf-g++ -std=c++11 -mcpu=arm1176jzf-s -mfpu=vfp \
	-o minimal minimal.cc model_utils.cc \
	-I$(MAKEFILE_DIR)/../../../ \
	-I$(MAKEFILE_DIR)/../../../libedgetpu/ \
	-I$(TENSORFLOW_DIR) \
	-I$(TENSORFLOW_DIR)/tensorflow/lite/tools/make/downloads/flatbuffers/include \
	-L$(TENSORFLOW_DIR)/tensorflow/lite/tools/make/gen/rpi_armv6/lib \
	-L$(TENSORFLOW_DIR)/edgetpu/libedgetpu/direct/armv6/ \
	-ltensorflow-lite -l:libedgetpu.so.1.0 \
	-lstdc++ -lpthread -lm -lz -ldl

clean:
	rm -f minimal
