CXX ?= g++

HL_MODULES = daubechies_x.o inverse_daubechies_x.o haar_x.o inverse_haar_x.o

ifdef BUILD_PREFIX
LIB_HALIDE = bin/$(BUILD_PREFIX)/libHalide.a
else
LIB_HALIDE = bin/libHalide.a
endif

LIBPNG_LIBS_DEFAULT = $(shell libpng-config --ldflags)
LIBPNG_CXX_FLAGS ?= $(shell libpng-config --cflags)
# Workaround for libpng-config pointing to 64-bit versions on linux even when we're building for 32-bit
ifneq (,$(findstring -m32,$(CXX)))
ifneq (,$(findstring x86_64,$(LIBPNG_LIBS_DEFAULT)))
LIBPNG_LIBS ?= -lpng
endif
endif
LIBPNG_LIBS ?= $(LIBPNG_LIBS_DEFAULT)
PNGFLAGS=$(LIBPNG_LIBS) $(LIBPNG_CXX_FLAGS)

ifeq ($(HL_TARGET),ptx)
  CUDA_LDFLAGS=-L/usr/lib/nvidia-current -lcuda
endif
ifeq ($(HL_TARGET),ptx-debug)
  CUDA_LDFLAGS=-L/usr/lib/nvidia-current -lcuda
endif
ifneq (, $(findstring opencl,$(HL_TARGET)))
  OPENCL_LDFLAGS=-lOpenCL
endif
ifneq (, $(findstring spir,$(HL_TARGET)))
  OPENCL_LDFLAGS=-lOpenCL
endif
ifneq (, $(findstring spir64,$(HL_TARGET)))
  OPENCL_LDFLAGS=-lOpenCL
endif

all: filter

wavelet: ../../ wavelet.cpp
	$(MAKE) -C ../../ $(LIB_HALIDE)
	$(CXX) wavelet.cpp -I ../../include ../../$(LIB_HALIDE) -o wavelet -lpthread -ldl

$(HL_MODULES): wavelet
	./wavelet

filter.o: filter.cpp $(HL_MODULES)
	$(CXX) $(PNGFLAGS) -I../support -O3 -c filter.cpp

filter: filter.o
	$(CXX) filter.o $(HL_MODULES) $(PNGFLAGS) $(CUDA_LDFLAGS) $(OPENCL_LDFLAGS) -o filter -lpthread

clean:
	rm -f wavelet filter filter.o $(HL_MODULES) $(HL_MODULES:%.o) $(HL_MODULES:%.o=%.h)

test: filter
	./filter ../images/gray.png
