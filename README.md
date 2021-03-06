## SkimCaffe Specific Description

A Caffe branch for training sparse CNN that provides 80-95% sparsity in
convolution and fully-connected layers (tested with AlexNet and GoogLeNet).
Our optimized sparse convolution and sparse-matrix-times-dense-matrix routines
effectively take advantage of the sparsity achieving ~3x speedups with 90%
sparsity (our sparse convolution routine will be migrated to libxsmm library so
that it can be also used by other frameworks like TensorFlow).
SkimCaffe also has experimental features of getting sparsity in Winograd
(L1_Winograd regularization) and we have prelimiary results for AlexNet.
Please let us know if you're interested in this experimental feature.

SkimCaffe has been only tested with bvlc_reference_caffenet bvlc_googlenet, and
there could be places where things do not work if you use other networks.
Please let us know if you encounter such issues and share .prototxt of the
network you are using.
We will try our best to support it as well.
Eventually, our sparse CNN implementation should be general enough to handle
all kinds of networks.

We assume you have a recent Intel compiler and MKL installed.
Tested environments: (Intel compiler version 15.0.3.187 and boost 1.59.0)
We also assume you have a recent x86 CPU with AVX2 or AVX512 support.
Direct sparse convolution and sparse fully-connected layers is only tested for AlexNet.
More details on direct sparse convolution is described at: https://arxiv.org/abs/1608.01409

1) Set up Intel compiler environment (compilervars.sh or compilervars.csh)

2) Compile libxsmm:

```
make libxsmm
```

2) Build Caffe as usual

Additional options:
KNL=1 # compiles for Knights Landing
SSE=1 # compiles for SSE
If neither KNL nor SSE is defined, by default we compile for AVX2 (Haswell or later)

3) Test:

```
# Test sparse CaffeNet
bzip2 -d models/bvlc_reference_caffenet/logs/acc_57.5_0.001_5e-5_ft_0.001_5e-5/0.001_5e-05_0_1_0_0_0_0_Sun_Jan__8_07-35-54_PST_2017/caffenet_train_iter_640000.caffemodel.bz2
env OMP_NUM_THREADS=16 KMP_AFFINITY=granularity=fine,compact,1 build/tools/caffe.bin test -model models/bvlc_reference_caffenet/test_direct_sconv_mkl.prototxt -weights models/bvlc_reference_caffenet/logs/acc_57.5_0.001_5e-5_ft_0.001_5e-5/0.001_5e-05_0_1_0_0_0_0_Sun_Jan__8_07-35-54_PST_2017/caffenet_train_iter_640000.caffemodel # assume you have 16 cores. Adjust OMP_NUM_THREADS variable accordingly for your number of cores

# Test sparse GoogLeNet
bzip2 -d models/bvlc_googlenet/gesl_0.686639_0.001_0.00005_ft_0.001_0.0001.caffemodel.bz2
env OMP_NUM_THREADS=16 KMP_AFFINITY=granularity=fine,compact,1 build/tools/caffe.bin test -model models/bvlc_googlenet/test_direct_sconv.prototxt -weights models/bvlc_googlenet/gesl_0.686639_0.001_0.00005_ft_0.001_0.0001.caffemodel
```

Example output from Intel(R) Xeon(R) CPU E5-2699 v4 @ 2.20GHz
Used 88 omp threads (using hyper-threading) and batch size 264

```
I0112 19:44:17.246243 83804 conv_relu_pool_lrn_layer.cpp:749] conv2 K-cycles-per-file max 203.198 avg 201.927 mFlops-per-file 447.898 GF/s 4838.16
I0112 19:44:17.255386 83804 conv_relu_layer.cpp:217] conv3 wall clock-time 0.00903296 padding-time 0.000645399 relu-time 9.89437e-05
I0112 19:44:17.255430 83804 conv_relu_layer.cpp:227] conv3 K-cycles-per-file max 68.5256 avg 66.8225 mFlops-per-file 299.041 GF/s 9578.53
I0112 19:44:17.263487 83804 conv_relu_layer.cpp:217] conv4 wall clock-time 0.00803304 padding-time 0.00048995 relu-time 0.000106096
I0112 19:44:17.263531 83804 conv_relu_layer.cpp:227] conv4 K-cycles-per-file max 59.3055 avg 56.6332 mFlops-per-file 224.281 GF/s 8300.77
I0112 19:44:17.273319 83804 conv_relu_pool_layer.cpp:342] conv5 wall clock-time 0.009758 pool-time 0.00255013
I0112 19:44:17.273363 83804 conv_relu_pool_layer.cpp:368] conv5 K-cycles-per-file max 52.2832 avg 51.2715 mFlops-per-file 149.52 GF/s 6277.1
I0112 19:44:17.279690 83804 inner_product_relu_dropout_layer.cpp:228] csrmm takes 0.00629187 effective GF/s 3167.79 real GF/s 308.281
I0112 19:44:17.284083 83804 inner_product_relu_dropout_layer.cpp:228] csrmm takes 0.00371099 effective GF/s 2387.07 real GF/s 373.345
I0112 19:44:17.286274 83804 inner_product_layer.cpp:223] csrmm takes 0.00164509 effective GF/s 1314.63 real GF/s 344.16
I0112 19:44:17.289224 83804 net.cpp:654]  Test time of data     29.689 ms ( 16.2876 % )
I0112 19:44:17.289305 83804 net.cpp:654]  Test time of label_data_1_split       0.002 ms ( 0.00109721 % )
I0112 19:44:17.289350 83804 net.cpp:654]  Test time of conv1    60.57 ms ( 33.2291 % )
I0112 19:44:17.289393 83804 net.cpp:654]  Test time of relu1    5.189 ms ( 2.84672 % )
I0112 19:44:17.289409 83804 net.cpp:654]  Test time of pool1    5.894 ms ( 3.23349 % )
I0112 19:44:17.289482 83804 net.cpp:654]  Test time of norm1    6.289 ms ( 3.45019 % )
I0112 19:44:17.289535 83804 net.cpp:654]  Test time of conv2    31.84 ms ( 17.4676 % )
I0112 19:44:17.289557 83804 net.cpp:654]  Test time of conv3    9.093 ms ( 4.98848 % )
I0112 19:44:17.289572 83804 net.cpp:654]  Test time of conv4    8.092 ms ( 4.43932 % )
I0112 19:44:17.289587 83804 net.cpp:654]  Test time of conv5    9.819 ms ( 5.38677 % )
I0112 19:44:17.289603 83804 net.cpp:654]  Test time of fc6      6.97 ms ( 3.82379 % )
I0112 19:44:17.289619 83804 net.cpp:654]  Test time of fc7      4.253 ms ( 2.33322 % )
I0112 19:44:17.289638 83804 net.cpp:654]  Test time of fc8      1.743 ms ( 0.956221 % )
I0112 19:44:17.289654 83804 net.cpp:654]  Test time of fc8_fc8_0_split  0.001 ms ( 0.000548607 % )
I0112 19:44:17.289669 83804 net.cpp:654]  Test time of accuracy 2.192 ms ( 1.20255 % )
I0112 19:44:17.289685 83804 net.cpp:654]  Test time of loss     0.644 ms ( 0.353303 % )
I0112 19:44:17.289746 83804 caffe.cpp:330] Total forwarding time: 182.28 ms
I0112 19:44:17.660748 83804 caffe.cpp:333] Loss: 1.83185
I0112 19:44:17.660802 83804 caffe.cpp:345] accuracy = 0.573712
I0112 19:44:17.660838 83804 caffe.cpp:345] loss = 1.83185 (* 1 = 1.83185 loss)
I0112 19:44:17.660853 83804 caffe.cpp:350] Total-images-processed: 13200
I0112 19:44:17.660862 83804 caffe.cpp:353] conv2 K-cycles-per-file 206.582 mFlops-per-file 447.898 GF/s 4758.9
I0112 19:44:17.660883 83804 caffe.cpp:353] conv3 K-cycles-per-file 74.764 mFlops-per-file 299.041 GF/s 8779.24
I0112 19:44:17.660899 83804 caffe.cpp:353] conv4 K-cycles-per-file 65.242 mFlops-per-file 224.281 GF/s 7545.37
I0112 19:44:17.660913 83804 caffe.cpp:353] conv5 K-cycles-per-file 57.779 mFlops-per-file 149.52 GF/s 5679.97
I0112 19:44:17.660930 83804 caffe.cpp:353] fc6 K-cycles-per-file 55.684 mFlops-per-file 75.4975 GF/s 2975.93
I0112 19:44:17.660943 83804 caffe.cpp:353] fc7 K-cycles-per-file 34.338 mFlops-per-file 33.5544 GF/s 2144.81
I0112 19:44:17.660958 83804 caffe.cpp:353] fc8 K-cycles-per-file 14.645 mFlops-per-file 8.192 GF/s 1227.76
```

# Caffe

[![Build Status](https://travis-ci.org/BVLC/caffe.svg?branch=master)](https://travis-ci.org/BVLC/caffe)
[![License](https://img.shields.io/badge/license-BSD-blue.svg)](LICENSE)

Caffe is a deep learning framework made with expression, speed, and modularity in mind.
It is developed by the Berkeley Vision and Learning Center ([BVLC](http://bvlc.eecs.berkeley.edu)) and community contributors.

Check out the [project site](http://caffe.berkeleyvision.org) for all the details like

- [DIY Deep Learning for Vision with Caffe](https://docs.google.com/presentation/d/1UeKXVgRvvxg9OUdh_UiC5G71UMscNPlvArsWER41PsU/edit#slide=id.p)
- [Tutorial Documentation](http://caffe.berkeleyvision.org/tutorial/)
- [BVLC reference models](http://caffe.berkeleyvision.org/model_zoo.html) and the [community model zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo)
- [Installation instructions](http://caffe.berkeleyvision.org/installation.html)

and step-by-step examples.

[![Join the chat at https://gitter.im/BVLC/caffe](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/BVLC/caffe?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Please join the [caffe-users group](https://groups.google.com/forum/#!forum/caffe-users) or [gitter chat](https://gitter.im/BVLC/caffe) to ask questions and talk about methods and models.
Framework development discussions and thorough bug reports are collected on [Issues](https://github.com/BVLC/caffe/issues).

Happy brewing!

## License and Citation

Caffe is released under the [BSD 2-Clause license](https://github.com/BVLC/caffe/blob/master/LICENSE).
The BVLC reference models are released for unrestricted use.

Please cite Caffe in your publications if it helps your research:

    @article{jia2014caffe,
      Author = {Jia, Yangqing and Shelhamer, Evan and Donahue, Jeff and Karayev, Sergey and Long, Jonathan and Girshick, Ross and Guadarrama, Sergio and Darrell, Trevor},
      Journal = {arXiv preprint arXiv:1408.5093},
      Title = {Caffe: Convolutional Architecture for Fast Feature Embedding},
      Year = {2014}
    }
