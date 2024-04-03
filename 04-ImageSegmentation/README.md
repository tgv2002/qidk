# Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
    + [About "Image Segmentation"](#about-image-segmentation)
    + [Pre-Requisites](#pre-requisites)
- [Model Selection and Conversion](#model-selection-and-conversion)
    + [Model Overview](#model-overview)
    + [Model Conversion](#model-conversion)
- [Source Overview](#source-overview)
    + [Source Organization](#source-organization)
    + [Code Implementation](#code-implementation)
- [Build APK file with Android Studio](#build-apk-file-with-android-studio)
- [Results](#results)

# Introduction

### About "Image Segmentation" 

- Current project is an sample Android application for AI-based Image Segmentation using [Qualcomm® AI Engine Direct](https://docs.qualcomm.com/bundle/publicresource/topics/80-63442-50/introduction.html) framework.
- We have used 4 Models in this Solution
- This sample segments objects in the Image.
- If users intend to use a different model in this demo framework, **image pre/post processing will be needed**. 
- Current pre/post processing is specific to the models used. 

### Pre-Requisites 

- Qualcomm® AI Engine Direct setup should be completed by following the guide [here](https://docs.qualcomm.com/bundle/publicresource/topics/80-63442-50/setup.html)
- Install onnx and onnxruntime using `pip install onnx onnxruntime`
- Android Studio to import sample project
- Android NDK to build native code
- Install opencv using ```pip install opencv-python```

# Model Selection

### Model Overview

Please refer to [Models](https://github.qualcomm.com/qualcomm-model-zoo-public-mirror/models-for-solutions/tree/main/04-image-segmentation) repository for model overview.

### Model Conversion

For the required model `<model_name>` implemented in the `models` folder

- Execute the corresponding `.ipynb` file to obtain the model library and serialized binary files. In case of error in execution, execute the instructions in the notebook on terminal.
- After execution, the model's .so file can be obtained from `models\<model_name>\models\model_libs2\aarch64-android` and serialized binary file can be obtained from `models\<model_name>\output` folder.
- Copy both the model files to `app\src\main\assets` and `app\src\main\jniLibs\arm64-v8a` folders, for utilizing in application.

# Source Overview

### Source Organization

- demo: Contains demo video, GIF 
- app: Contains source files in standard Android app format.
- app\src\main\assets, app\src\main\jniLibs\arm64-v8a : Contains Model library file / cached binary
- app\src\main\java\com\qcom\imageSegmentation : Application java source code
- app\src\main\cpp : Application C++(native) source code
- sdk : Contains openCV sdk (Will be generated using _ResolveDependencies.sh_ )
   
### Code Implementation

- Model Initialization
   
   `public boolean loadingMODELS(char runtime_var, String model_name)`
    - runtime_var: Name of the backend. Possible options are "libQnnCpu.so", "libQnnHtp.so".
    - model_name: File name of the stored model. Based on the backend, it is loaded dynamically.
  
- Running Model

    - Following is the Java Function, that handles model execution. This function iternally calls sub functions to handle pre-processing and post-processing

      `inferQNN(inputMat.getNativeObjAddr(), outputMat.getNativeObjAddr())`
        - inputMat is opencv Matrix that contains input image.
        - outputMat is the destination for the output image

   - C++ function that handles preprocessing for the input image.
   
       `void preprocess(cv::Mat &img)`
  
   - Model gives a segmented map that we can see in executeModel function
      
       `bool executeModel(cv::Mat &img, int orig_width, int orig_height, float &milli_time, cv::Mat &destmat)`

  - QNN API function that runs the network and give result

    `qnn->execute(inputMap, outputMap);`



# Build APK file with Android Studio  

1. Clone this repo.
2. Set the environment variable `QNN_SDK_ROOT` by running the command `export QNN_SDK_ROOT=<QNN_SDK_PATH_HERE>`, suitably replacing the placeholder. After that, execute `bash resolveDependencies.sh`.
    * This script will download opencv and paste to sdk directory, to enable OpenCv for android Java.
    * This script will copy all necessary library files to `app\src\main\jniLibs\arm64-v8a` for execution.
3. Generate model files using the steps mentioned.
4. Import folder ImageSegmentation as a project in Android Studio 
5. Do gradle sync
6. Compile the project. 
7. Output APK file should get generated : app-debug.apk
8. Prepare the Qualcomm Innovators development kit(QIDK) to install the application (Do not run APK on emulator)
9. Install and test application : app-debug.apk

```java
adb install -r -t app-debug.apk
```

10. launch the application


###### *Qualcomm Neural Processing SDK and Snapdragon are products of Qualcomm Technologies, Inc. and/or its subsidiaries. AIMET Model Zoo is a product of Qualcomm Innovation Center, Inc.*
