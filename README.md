OpenPose 开源姿态工程
====================================

##  介绍

OpenPose是一个**实时多人关键点检测库，是用C++的多线程技术写成的**，同时也使用了OpenCV 和 Caffe*
* 用Caffe训练的，但是准备提供用于tensorflow 和  Torch的接口.（如果你实现了这些接口，欢迎融合进来).OpenPose 可免费用于非商业应用。如果用作商业应用，请联系我们。 

本库的主要功能：
* 多人 15 或 **18身体姿态关键点**

* 多人 **2x21手部关键点**

* 多人 **70-key-point face** 

* **多线程** 模块

* 可图片，可视频，可摄像头

* 结果兼容各种格式（JSON,XML,PNG,JPG,...)

* 可视化简单

* 所有的功能都被融合进了：  **simple-to-use OpenPose wrapper class**.

本姿态估计工作是基于来自[the ECCV 2016 demo](https://github.com/CMU-Perceptual-Computing-Lab/caffe_rtpose), "Realtime Multiperson Pose Estimation"的C++代码, [Zhe Cao](http://www.andrew.cmu.edu/user/zhecao), [Tomas Simon](http://www.cs.cmu.edu/~tsimon/), [Shih-En Wei](https://scholar.google.com/citations?user=sFQD3k4AAAAJ&hl=en), [Yaser Sheikh](http://www.cs.cmu.edu/~yaser/). The [full project repo](https://github.com/ZheC/Multi-Person-Pose-Estimation) includes Matlab and Python version, as well as training code.全部工程包含 matlab和python代码。



## 结果展示

### 身体关键点
<p align="center">
    <img src="doc/media/dance.gif", width="480">
</p>

## Coming Soon (But Already Working!)

### 身体+手+脸
<p align="center">
    <img src="doc/media/pose_face_hands.gif", width="480">
</p>

### 身体 + 脸 
<p align="center">
    <img src="doc/media/pose_face.gif", width="480">
</p>

### 身体 + 手
<p align="center">
    <img src="doc/media/pose_hands.gif", width="480">
</p>


## Contents 内容列表
1. [安装](#安装)
2. [快速开始](#快速开始)
    1. [Demo](#demo)
    2. [OpenPose Wrapper](#openpose-wrapper)
    3. [OpenPose Library](#openpose-library)
3. [输出](#输出)
    1. [Output Format](#output-format)
    2. [Reading Saved Results](#reading-saved-results)
4. [Send Us your Feed-Back!](#send-us-your-feed-back)
5. [Citation](#citation)



## 安装
Installation steps on [doc/installation.md](doc/installation.md).



## 快速开始
Most users cases should not need to dive deep into the library, they might just be able to use the [Demo](#demo) or the simple [OpenPose Wrapper](#openpose-wrapper). So you can most probably skip the library details on [OpenPose Library](#openpose-library).



#### Demo
Your case if you just want to process a folder of images or video or webcam and display or save the pose results.

Forget about the OpenPose library details and just read the [doc/demo_overview.md](doc/demo_overview.md) 1-page section.

#### OpenPose Wrapper
Your case if you want to read a specific format of image source and/or add a specific post-processing function and/or implement your own display/saving.

(Almost) forget about the library, just take a look to the `Wrapper` tutorial on [examples/tutorial_wrapper/](examples/tutorial_wrapper/).

Note: you should not need to modify OpenPose source code or examples, so that you can directly upgrade the OpenPose library anytime in the future without changing your code. You might create your custom code on [examples/user_code/](examples/user_code/) and compile it by using `make all` in the OpenPose folder.

#### OpenPose Library
Your case if you want to change internal functions and/or extend its functionality. First, take a look to the [Demo](#demo) and [OpenPose Wrapper](#openpose-wrapper). Secondly, read the 2 following subsections: OpenPose Overview and Extending Functionality.

1. OpenPose Overview: Learn the basics about our library source code on [doc/library_overview.md](doc/library_overview.md).

2. Extending Functionality: Learn how to extend our library on [doc/library_extend_functionality.md](doc/library_extend_functionality.md).

3. Adding An Extra Module: Learn how to add an extra module on [doc/library_add_new_module.md](doc/library_add_new_module.md).

#### Doxygen Documentation Autogeneration
You can generate the documentation by running the following command. The documentation will be generated on `doc/doxygen/html/index.html`. You can simply open it with double click (your default browser should automatically display it).
```
cd doc/
doxygen doc_autogeneration.doxygen
```



## 输出
#### Output Format
There are 2 alternatives to save the **(x,y,score) body part locations**. The `write_pose` flag uses the OpenCV cv::FileStorage default formats (JSON, XML and YML). However, the JSON format is only available after OpenCV 3.0. Hence, `write_pose_json` saves the people pose data as a custom JSON file. For the later, each JSON file has a `people` array of objects, where each object has an array `body_parts` containing the body part locations and detection confidence formatted as `x1,y1,c1,x2,y2,c2,...`. The coordinates `x` and `y` can be normalized to the range [0,1], [-1,1], [0, source size], [0, output size], etc., depending on the flag `scale_mode`. In addition, `c` is the confidence in the range [0,1].

```
{
    "version":0.1,
    "people":[
        {"body_parts":[1114.15,160.396,0.846207,...]},
        {"body_parts":[...]},
    ]
}
```

The body part order of the COCO (18 body parts) and MPI (15 body parts) keypoints is described for `POSE_BODY_PART_MAPPING` in [include/openpose/pose/poseParameters.hpp](include/openpose/pose/poseParameters.hpp). E.g. for COCO:
```
    POSE_COCO_BODY_PARTS {
        {0,  "Nose"},
        {1,  "Neck"},
        {2,  "RShoulder"},
        {3,  "RElbow"},
        {4,  "RWrist"},
        {5,  "LShoulder"},
        {6,  "LElbow"},
        {7,  "LWrist"},
        {8,  "RHip"},
        {9,  "RKnee"},
        {10, "RAnkle"},
        {11, "LHip"},
        {12, "LKnee"},
        {13, "LAnkle"},
        {14, "REye"},
        {15, "LEye"},
        {16, "REar"},
        {17, "LEar"},
        {18, "Bkg"},
    }
```

For the **heat maps storing format**, instead of individually saving each of the 67 heatmaps (18 body parts + background + 2 x 19 PAFs) individually, the library concatenate them vertically into a huge (width x #heat maps) x (height) matrix, i.e. it concats the heat maps by columns. E.g. columns [0, individual heat map width] contains the first heat map, columns [individual heat map width + 1, 2 * individual heat map width] contains the second heat map, etc. Note that some displayers are not able to display the resulting images given its size. However, Chrome and Firefox are able to properly open them.

The saving order is body parts + background + PAFs. Any of them can be disabled with the program flags. If background is disabled, then the final image will be body parts + PAFs. The body parts and background follow the order of `POSE_COCO_BODY_PARTS` or `POSE_MPI_BODY_PARTS`, while the PAFs follow the order specified on POSE_BODY_PART_PAIRS in `poseParameters.hpp`. E.g. for COCO:
```
    POSE_COCO_PAIRS    {1,2,   1,5,   2,3,   3,4,   5,6,   6,7,   1,8,   8,9,   9,10, 1,11,  11,12, 12,13,  1,0,   0,14, 14,16,  0,15, 15,17,   2,16,  5,17};
```

Where each index is the key value corresponding with each body part on `POSE_COCO_BODY_PARTS`, e.g. 0 for "Neck", 1 for "RShoulder", etc.

#### Reading Saved Results
We use standard formats (JSON, XML, PNG, JPG, ...) to save our results, so there will be lots of frameworks to read them later, but you might also directly use our functions on [include/openpose/filestream.hpp](include/openpose/filestream.hpp). In particular, `loadData` (for JSON, XML and YML files) and `loadImage` (for image formats such as PNG or JPG) to load the data into cv::Mat format.



## Custom Caffe
We only modified some Caffe compilation flags and minor details. You can use use your own Caffe distribution, these are the files we added and modified:

1. Added files: `install_caffe.sh`; as well as `Makefile.config.Ubuntu14.example`, `Makefile.config.Ubuntu16.example`, `Makefile.config.Ubuntu14_cuda_7.example` and `Makefile.config.Ubuntu16_cuda_7.example` (extracted from `Makefile.config.example`). Basically, you must enable cuDNN.
2. Edited file: Makefile. Search for "# OpenPose: " to find the edited code. We basically added the C++11 flag to avoid issues in some old computers.
3. Optional - deleted Caffe file: `Makefile.config.example`.
4. Finally, run `make all && make distribute` in your Caffe version and modify the Caffe directory variable in our Makefile config file: `./Makefile.config.UbuntuX.example` (where X is 14 or 16 depending on your Ubuntu version), set the `CAFFE_DIR` parameter to the path where both the `include` and `lib` Caffe folders are located.



## Send Us your Feed-Back!
Our library is open source for research purposes, and we want to continuously improve it! So please, let us know if...

1. ... you find any bug (in functionality or speed).

2. ... you added some functionality to some class or some new Worker<T> subclass which we might potentially incorporate to our library.

3. ... you know how to speed up or make more clear any part of the library.

4. ... you have request about possible functionality.

5. ... etc.

Just comment on GibHub or make a pull request! We will answer you back as soon as possible!



## Citation
Please cite the paper in your publications if it helps your research:

### Pose Estimation

    @inproceedings{cao2017realtime,
      author = {Zhe Cao and Tomas Simon and Shih-En Wei and Yaser Sheikh},
      booktitle = {CVPR},
      title = {Realtime Multi-Person 2D Pose Estimation using Part Affinity Fields},
      year = {2017}
      }

    @inproceedings{simon2017hand,
      author = {Tomas Simon and Hanbyul Joo and Iain Matthews and Yaser Sheikh},
      booktitle = {CVPR},
      title = {Hand Keypoint Detection in Single Images using Multiview Bootstrapping},
      year = {2017}
      }

    @inproceedings{wei2016cpm,
      author = {Shih-En Wei and Varun Ramakrishna and Takeo Kanade and Yaser Sheikh},
      booktitle = {CVPR},
      title = {Convolutional pose machines},
      year = {2016}
      }
