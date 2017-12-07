# RAIL Object Detector

## Two Minute Intro

This detector uses [Deformable R-FCN](https://github.com/msracver/Deformable-ConvNets) to perform object detection. It provides the ability to query for objects in an image through a topic.

The response to all queries contains a list of objects, each of which has the following properties:

1. `label` - Object label
1. `probability` - confidence value in recognition
1. `centroid_x` - X pixel value of the centroid of the bounding box
1. `centroid_y` - Y pixel value of the centroid of the bounding box
1. `left_bot_x` - X pixel value of bottom-left corner of bounding box
1. `left_bot_y` - Y pixel value of bottom-left corner of bounding box
1. `right_top_x` - X pixel value of top-right corner of bounding box
1. `right_top_x` - X pixel value of top-right corner of bounding box

#### Fetching Object Detections

The detector subscribes to an existing camera sensor topic and grabs images from this camera at predetermined intervals. After performing object detection on the grabbed image, the detector publishes the list of objects that were found to the query topic, and re-uses the header of the input image.

The interval for grabbing images is specified in the form of a frequency. If the desired frequency exceeds the maximum frequency of operation of the detector (~1 Hz on CPU, ~4 Hz on Titan X), we limit to the maximum frequency of operation.

## Menu
 * [Requirements](#requirements)
 * [Installation](#installation)
 * [Testing your Installation](#testing-your-installation)
 * [ROS Nodes](#ros-nodes)
 * [Startup](#startup)
 * [Building with CUDA](#building-with-cuda)
 * [Scope for Improvement](#scope-for-improvement)

## Requirements

Below is a list of python libraries that are required for the Deformable RFCN network in MXNet to work:
 * Cython
 * EasyDict
 * opencv-python
 * mxnet-cu80==0.12.0b20171027
 * Pillow
 * pyyaml
 
Note that the version of MXNet is the same as in the Deformable ConvNets repo. You may change the version if your setup requires something different, but it may not work properly.

There is a `requirements.txt` file included in the repo, which lists the above libraries. You can install all of them at once by running:
```
pip install -r requirements.txt
```

## Installation

1. Put this package into your workspace
1. Navigate to the top level of this package (where this README is located)
1. run `sh init.sh`
1. Download the model parameters from [this Dropbox link](https://www.dropbox.com/s/b5n215zidzqaxft/rfcn_dcn_coco-0000.params?dl=0) and move them into the `libs/model` subdirectory of this package.
1. Run `catkin_make` and enjoy the ability to use object detection! (If you need to update the file paths, use absolute paths!)

## Testing your Installation

There is a testing launch file and script to make sure you have everything together and running smoothly. First, add some `.jpg` images to `rail_object_detector/libs/darknet/data`. Then run:
```
roslaunch rail_object_detector def_detector_demo.launch
```
In a separate terminal, run:
```
rosrun image_view image_view image:=/rail_detector_node/debug/object_image
```
and you will see the image you pointed to with detected objects highlighted and labeled. It should look something like this:

![Visualization of the object detector](objects.gif)

Colors change with each new detection of the object, and note that there is no tracking or propagation of labels (as on the couch in the gif above).


 You can also launch the demo node with a publish rate parameter, which slows or speeds up the cycling of images in the `libs/darknet/data` directory. For example, to publish at 0.5 Hz I could run:
```
roslaunch rail_object_detector def_detector_demo.launch rate:=0.5
```

Otherwise, you can run a camera with whichever ROS camera node you would like.  
1. Run a camera with your favorite ROS camera node.
1. Launch the `detector_node` node with the image topic of your camera and debug mode enabled:
```
roslaunch rail_object_detector def_detector.launch image_sub_topic_name:=[camera topic here] debug:=true`
```
Once again, in a separate terminal, run:
```
rosrun image_view image_view image:=/rail_object_detector/debug/object_image
```
and you will see the image you pointed to with detected objects highlighted and labeled.

## ROS Nodes

### detector_node

Wrapper for object detection through ROS services.  Relevant parameters are as follows:

* **Topics**
  * `detector_node/detections` ([object_detector/Detections](msg/Detections.msg))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Topic with object detections performed in the background by grabbing images at a specified interval.
  * `detector_node/debug/object_image` ([object_detector/debug/object_image])
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Topic with object detections visualized on incoming images as they come in from the subscriber. Only published if `debug:=true`.
* **Parameters**
  * `image_sub_topic_name` (`string`, default: "/kinect/qhd/image_color_rect")
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Image topic name to subscribe to for object detection
  * `debug` (`bool`, default: false)
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Enable or disable debug mode, which publishes input images with object bounding boxes and labels overlaid
  * `use_compressed_image` (`bool`, default: false)
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Flag to use the compressed version of your input image stream. It will append "\_compressed" to the name of your image topic, and changes how the input images are read (should just work though)

## Startup

Simply run the launch file to bring up all of the package's functionality:
```
roslaunch rail_object_detector def_detector.launch
```

## Scope for Improvement

There are a few ways to improve this package which we haven't yet tackled. Namely:

1. Include the ability to download the weights files automatically from the `CMakeLists.txt` file
1. There is plenty of room for better logging - I do most of mine using the debugger, so there aren't as many status print commands as normal
1. There is a distinct lack of defensive programming against malicious (NULL) messages and the like. Beware.

The immediately TODO list includes:
- [x] Create a demo publisher to cycle through images
- [x] Remove absolute paths
- [ ] Move `libs/darknet/data` to `libs/data` for clarity / ubiquity
- [ ] Add a service call for the deformable convnets
- [ ] Remove the need to run `sh init.sh`