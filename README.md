
# Sensor Fusion for Autonomous Vehicle using Lidar and Camera Sensors

This project aims to combine data from two sensors, cameras (2D) and LiDAR(3D), into a real time obstacle tracking and classication model for AVs. This is based on data collected from cars fitted with cameras and Velodyne LiDAR systems, and the config files for the setup are included in this repo. The output of this project is shown below.

https://github.com/user-attachments/assets/7f0f6eb6-fb91-4a40-a286-22ee8cbc4606

## What about the data?

The data here comes from RGB cameras mounted on the car (images), as well as a LiDAR system fitted on the top that outputs point clouds. You can find the files in the data folder, separated into images & points.

## What's the idea behind this?

As autonomous vehicles become more mainstream, it's fun to see how they perceive the world around them. Here's a typical setup in a rig like such:

![2](https://github.com/user-attachments/assets/41f6ac46-870b-4e89-93d3-09164acf64fc)

Overall, the goal is to combine the data from these sensors to make the machine perceive obstacles, track them and classify them. This can then be used downstream to decide ideal reactions to each different kind of obstacle object. The algorithm YOLOV4 is used in this project to track and classify the objects in 2D. 

The challeng here is to combine 3D point clouds from the Velodyne laser scanner with the 2D images coming from the cameras.
To solve this, the first step would be to project the points from the LiDAR scan onto the 2D pixels. The second part is then how do we take these sensors and draw 3D bounding boxes around obstacles.

## Three Most Important things about Sensor Fusion

In this Project, I have built my understanding around three most important things about Sensor Fusion:

1. Sensors and Sensor Fusion
2. Early Fusion
3. Late Fusion

# 1. Sensors and Sensor Fusion

In this project, my focus is on Cameras and LiDARs (IMAGES and POINT CLOUDS), which are being used by almost every Self Driving Car Company.

### Camera:
In below Picture, shows how Tesla's Autopilot car. And notice that we have 3 cameras, one in left, one in right and one in the center, side cameras are generally called stereo pairs which makes it easy to estimate disance to an obstacle.

![camera-housing](https://github.com/user-attachments/assets/f96213d5-98ba-43ca-896a-a50cec572686)

### LiDARs: (Light Detection and Ranging)

![lidar](https://github.com/user-attachments/assets/8cc41cb7-e598-4971-8521-f08cdbbbba34)

In above picture, shows the waymo's autonomous vehicle.
So, This is how LiDAR sensors work:

![lidar_working (1)](https://github.com/user-attachments/assets/ab4b0dcd-7ea4-4685-a508-e3df95c14fc6)

1.They send out a beam of light.
2.This light beam hits an object and bounces back.
3.LiDAR measures how long it takes for the light to bounce back.
4.Using the time it takes (time-of-flight), LiDAR calculates the distance to the object.

So, LiDAR uses the time it takes for light to hit something and come back to figure out how far away that thing is. This helps create detailed maps of the surroundings and is used in technologies like autonomous cars and mapping. 

### Point Clouds:
LiDAR generate point clouds. Point clouds are collections of points, and each point has its XYZ position. This allows us to accurately know the depth and distance of every obstacle or object in a 3D space.

Let's take a look at the following video to understand more about Point Clouds, These point clouds were captured with Basler Blaze 101 Time of Flight Camera. 

### Camera and LiDAR Fusion:
A self-driving car typically has:

Multiple Cameras: These cameras have different angles and capabilities. Some can see far with a narrow view, while others have a wider view but shorter range.

LiDARs: These sensors detect objects around the car and precisely estimate their 3D positions.

# 2. Early Fusion : First we fuse, then we detect

So, here is the Self Driving Car Setup from [KITTI Vision Benchmark Suite](https://www.cvlibs.net/datasets/kitti/setup.php).

This Self Driving Car setup has one Valodyne LiDAR (HDL-64E Laser Scanner), and a total of 4 cameras, 2 color cameras and 2 Grayscale cameras, Two cameras (Color and Gray) on left side, two cameras (Color and Gray) on right side. The sensors orientation is not the same, coordinate system is different and psition is also different. https://velodynelidar.com/blog/hdl-64e-lidar-sensor-retires/

### Projecting a LiDAR Point (3D) in a Camera Image (2D)

Considering an image, and a point cloud, and this is the output of projection!

And to do this projection, Information of physical position of sensors in needed, and how to convert from one to another.

So here is the list of Sensors we are using data of.

1. 1 X Velodyne HDL-64E Laserscanner
2. 4 X [FLIR Point Cameras](https://www.teledynevisionsolutions.com/products/flea3-usb3/?segment=iis&vertical=machine%20vision)
3. LiDAR and cameras use different systems to see and aren't in the same spot: it means that we're seeing an obstacle from 2 different positions with two different frames. We'll therefore need to convert the point seen by the LiDAR to the camera space, and then to the image space!

### Steps :
### 1. Project the Point Clouds (3D) to the Image (2D)
Convert the points from the LiDAR frame to the Image frame.
Projection Formula: Here is the formula to convert a point X in 3D into a point Y in 2D.

![formula](https://github.com/user-attachments/assets/84b2a1b3-33d9-428d-8765-84bc3f5fc54e)

# Y = P × R₀ × R|t × X

where:
- **X**: 3D coordinates  
- **R|t**: Rotation and translation matrix from the LiDAR to the Camera  
- **R₀**: Stereo vision rectifier  
- **P**: Intrinsic and extrinsic camera calibration parameters  

above formula is highly dependent on our sensor setup system.

### P:

A camera produces images by capturing light from the environment through its lens and converting it into a digital image. When buying a camera, it's typically pre-calibrated, meaning it's set up to produce images accurately without requiring you to adjust complex settings right away. This calibration ensures that the images you take are properly exposed and in focus.

Calibration is the process of teaching your camera how to translate a point in the real world (3D) into a pixel on the camera sensor. The intrinsic calibration matrix, the P, is a critical component in this process. It helps the camera understand the relationship between the 3D world and the 2D image it captures.

![mat](https://github.com/user-attachments/assets/6d6b1c7e-b8c6-4f6e-a0fd-4f1923975f6d)

In above matrix, f is is the focal length (fx, fy) and c is the optical center (cx, cy). This matrix is used to transform from the camera's perspective (camera frame) to the image's perspective (image frame).

### R0:

In stereo vision, the aim is to make the left and right images line up perfectly, it helps in drawing a horizontal line across them, below are the same objects aligned with each other.

![im01](https://github.com/user-attachments/assets/77a26c0c-2b7a-4a5e-9f04-75a2df4c9ec9)

We refer to that horizontal line as the "Epipolar Line."

![im_02](https://github.com/user-attachments/assets/098008c4-2cd3-4a7c-9db9-fe59de49fc3a)

In Stereo Vision, the matrix R0 aligns the left and right images, making them match perfectly. This step isn't necessary with a single camera, but it's crucial for stereo vision to work correctly.

### R0|t:

R|t is the transformation from the Velodyne frame of reference to the Camera frame of reference. It describes how data measured in the Velodyne's coordinate system can be translated and rotated to align with the coordinate system of the Camera.

![im03](https://github.com/user-attachments/assets/5902e946-f654-49d3-85e2-8991493c3e14)

![im04](https://github.com/user-attachments/assets/eea48b7d-56da-42df-9ccd-4a6fd198cec8)

In the above picture, Valedyne LiDAR Coordinate system is different from Camera Coordinate system.
So we have to go from Velodyne coordinate system to Camera coordinate system using coordinate transformation.

Keypoints:

1. To project a point obtained from a LiDAR in an image, we need to understand clearly how are our       sensors positioned, and what are their coordinate system.
2. The projection formula is Y = P * R0 * R|t * X
3. R|t is the matrix that convert a point from the Velodyne Frame to the Image frame. R0 is a matrix to rectify the stereo cameras. P is the intrinsic calibration matrix to take a point from the camera frame to the image (pixel).

### 2. Detect Obstacles in 2D (Camera)
This is what we do in object detection, given an image and output bounding boxes.
- Used a **YOLOv4** network  

### 3. Fuse the Results
- Shrink the bounding boxes  
- Filter the outliers using **one-sigma**  
- Retrieve the best distance using the **average**  


# 3. Late Fusion : First we detect, then we fuse

### Steps :
### 1. Detect Obstacles in 2D (Camera)
- Used a **YOLOv4** network  

### 2. Detect Obstacles in 3D (LiDAR)
- **Not implemented**: Assumed to have the position **(X, Y, Z)**  
- 3D bounding box: **(W, H, L)**  
- Orientation of the 3D bounding box: **Rᵧ**  

### 3. Project the 3D Obstacles in the Image
- Project points from 3D to 2D  
  - Convert to **homogeneous coordinates**  
  - Multiply by **P** (projection matrix)  
  - Convert to **Cartesian coordinates**  
- Compute the **3D bounding box** in the camera frame using:  
  **(X, Y, Z, W, H, L, Rᵧ)**  
- Project the **3D bounding box** in the camera frame to the image  
- Convert the **3D bounding box** to a **2D bounding box** and draw it  

### 4. Fuse the 3D and 2D Bounding Boxes
- Show the **LiDAR and Camera** boxes on the same image  
- Compute the **IOU (Intersection Over Union)** metrics for each box  
- Use the **Hungarian Algorithm** to match boxes based on the IOU matrix  
- Create the **final fused objects**  
    
# Results
The results are pretty fun to see! I'm attaching a few examples below. Some useful resources for the datasets are listed in the link below.
https://boschresearch.github.io/multimodalperception/dataset.html

Results from Part 1 (Projecting LiDAR onto Images):

![3](https://github.com/user-attachments/assets/b3fc2f69-7d22-4b8f-815c-3c690ab38353)

Results from Part 2 (Drawing 3D Bounding Boxes and keeping only points of interest):

![4](https://github.com/user-attachments/assets/7c8640f2-a949-481a-9972-f401dcf5c036)

Results on a highway dataset:

![5](https://github.com/user-attachments/assets/43031158-bdbc-4903-8faa-ba2ad9cc87cc)

As you might see, the results are pretty good!

