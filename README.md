# Sensor-Fusion
Fusing data from a LiDAR and a Camera. This is an example of the output of the early fusion algorithm:

https://github.com/sua_7777/Sensor-Fusion-for-Autonomous-Vehicle/blob/main/Desktop/LidarFusion/videos/out_3.mp4

<video width="700" controls>
  <source src="Desktop/LidarFusion/videos/out_3.mp4" type="video/mp4">
</video>

# Sensor Fusion for Autonomous Vehicle using Lidar and Camera Sensors

This project aims to combine data from two sensors, cameras (2D) and LiDAR(3D), into a real time obstacle tracking and classication model for AVs. This is based on data collected from cars fitted with cameras and Velodyne LiDAR systems, and the config files for the setup are included in this repo.

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

# Early Fusion : First we fuse, then we detect

### Steps :
### 1. Project the Point Clouds (3D) to the Image (2D)
Convert the points from the LiDAR frame to the Image frame.

#Y = P × R₀ × R|t × X
where:
- **X**: 3D coordinates  
- **R|t**: Rotation and translation matrix from the LiDAR to the Camera  
- **R₀**: Stereo vision rectifier  
- **P**: Intrinsic and extrinsic camera calibration parameters  

Select the points that are visible in the image.

### 2. Detect Obstacles in 2D (Camera)
- Used a **YOLOv4** network  

### 3. Fuse the Results
- Shrink the bounding boxes  
- Filter the outliers using **one-sigma**  
- Retrieve the best distance using the **average**  


# Late Fusion : First we detect, then we fuse

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
    
## Results
The results are pretty fun to see! I'm attaching a few examples below. Some useful resources for the datasets are listed in the link below.
https://boschresearch.github.io/multimodalperception/dataset.html

Results from Part 1 (Projecting LiDAR onto Images):
![3](https://github.com/user-attachments/assets/b3fc2f69-7d22-4b8f-815c-3c690ab38353)

Results from Part 2 (Drawing 3D Bounding Boxes and keeping only points of interest):
![4](https://github.com/user-attachments/assets/7c8640f2-a949-481a-9972-f401dcf5c036)

Results on a highway dataset:
![5](https://github.com/user-attachments/assets/43031158-bdbc-4903-8faa-ba2ad9cc87cc)

As you might see, the results are pretty good!

