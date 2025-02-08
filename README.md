# Seosor Fusion for Autonomous Vehicle using Lidar and Camera Sensors

This project aims to combine data from two sensors, cameras (2D) and LiDAR(3D), into a real time obstacle tracking and classication model for AVs. This is based on data collected from cars fitted with cameras and Velodyne LiDAR systems, and the config files for the setup are included in this repo.

## What about the data?

The data here comes from RGB cameras mounted on the car (images), as well as a LiDAR system fitted on the top that outputs point clouds. You can find the files in the data folder, separated into images & points.

## What's the idea behind this?

As autonomous vehicles become more mainstream, it's fun to see how they perceive the world around them. Here's a typical setup in a rig like such:

![687474703a2f2f7777772e63766c6962732e6e65742f64617461736574732f6b697474692f696d616765732f7061737361745f73656e736f72732e6a7067](https://github.com/user-attachments/assets/d447f8db-73cc-46e8-b143-68154dd55dca)

![2](https://github.com/user-attachments/assets/41f6ac46-870b-4e89-93d3-09164acf64fc)

Overall, the goal is to combine the data from these sensors to make the machine perceive obstacles, track them and classify them. This can then be used downstream to decide ideal reactions to each different kind of obstacle object. The algorithm YOLOV4 is used in this project to track and classify the objects in 2D. 

The challeng here is to combine 3D point clouds from the Velodyne laser scanner with the 2D images coming from the cameras.
To solve this, the first step would be to project the points from the LiDAR scan onto the 2D pixels. The second part is then how do we take these sensors and draw 3D bounding boxes around obstacles.

## Results
The results are pretty fun to see! I'm attaching a few examples below. Some useful resources for the datasets are listed in the link below.
https://boschresearch.github.io/multimodalperception/dataset.html

Results from Part 1 (Projecting LiDAR onto Images):
![3](https://github.com/user-attachments/assets/799d443a-9d4a-4e83-a366-0916b29d8a1d)

Results from Part 2 (Drawing 3D Bounding Boxes and keeping only points of interest):
![4](https://github.com/user-attachments/assets/3bffd27c-d485-44b8-8d9f-8e1fd404ce30)


Results on a highway dataset:
![5](https://github.com/user-attachments/assets/b6f3f07c-2f92-4c54-95f8-9fbee1c7d100)

As you might see, the results are pretty good!
