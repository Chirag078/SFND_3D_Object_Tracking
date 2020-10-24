# SFND 3D Object Tracking

The flowchart below provides an overview of the final project structure. The steps in the orange box were completed in the previous project [2D Feature Tracking](https://github.com/walkerbrown/sfnd-camera-2d). This project builds on that previous one, implementing the steps in the blue box and downstream. Objects detected with the YOLO deep neural network are tracked across frames by considering the strength of keypoint correspondences within their bounding boxes. Finally, a robust estimation of time-to-collision (TTC) is performed with data from both the lidar and camera sensors.

<img src="images/course_code_structure.png" width="779" height="414" />

## Dependencies
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* Git LFS
  * Weight files are handled using [LFS](https://git-lfs.github.com/)
  * `brew install git-lfs  # Install for macOS, see link above for others`
  * `git remote add https://github.com/udacity/SFND_3D_Object_Tracking.git`
  * `git lfs install`
  * `git lfs pull upstream`
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Building and running the project
```
mkdir build && cd build
cmake ..
make
./3D_object_tracking
```
## Concepts 
- Keypoint detectors and descriptors
- Object detection using the pre-trained YOLO deep-learning framework
- Methods to track objects by matching keypoints and bounding boxes across successive images
- Associating regions in a camera image with lidar points in 3D space

## Tasks
### FP.1 Match 3D objects

 - `std::multimap<int,int>` is used to track pairs of bounding box IDs. 
 - counted the keypoint correspondences per box pair to determine the best matches between frames.
 - Refer Lines from 213-256 in camFusion_Student.cpp


### FP.2 Compute lidar-based TTC

 - In each frame, the median x-distance taken to reduce the impact of outlier lidar points on TTC estimate. 
 - With the constant velocity model, We can use the below equation.
```
TTC = y * (1.0 / frameRate) / (x - y);
```
 - Refer Lines 201-210 in camFusion_Student.cpp 

### FP.3 Associate keypoint matches with bounding boxes
  - clusterKptMatchesWithROI is implimented to associate keypoints matches with bouding box
  -  it loops through every matched keypoint pair in an image. If the keypoint falls within the bounding box region-of-interest (ROI) in the current frame, the keypoint match is associated with the current `BoundingBox` data structure.
  - Refer Lines from 138-147 in camFusion_Student.cpp

### FP.4 Compute mono camera-based TTC
 
 -  computeTTCCamera function uses distance ratios on keypoints matched between frames to determine the rate of scale change within an image. 
 - This rate of scale change can be used to estimate the TTC as shown in below equation.
```
TTC = (-1.0 / frameRate) / (1 - medianDistRatio);
```
 - Refer Lines from 151-188 in camFusion_Student.cpp

### FP.5 Performance evaluation, lidar outliers
 - Lidar estimated TTC ranged from about 8-15 seconds
 - Approach of taking the median point, rather than the closest point, has avoided the problem introduced by outliers. 

### FP.6 Performance evaluation, detector/descriptor combinations
 - Certain detector/descriptor combinations, especially the `Harris` and `ORB` detectors, produced very unreliable camera TTC estimates. 
 - `SIFT`, `FAST`, and `AKAZE` detectors produced reliable results in line with the TTC estimates provided by the lidar sensor alone. 
 - Refer the output.xlsx file for performance evaluation