This code is a part of our Final year project "Programming and Visualization of SCARA robot".

In this code, we detect the position and dimensions of the object and find centroid of the object using open cv.
The object is placed on the conveyor belt. After scanning the image from camera, this code gives the coordinates of the object and it's centroid. This coordinates are applied as input to the reverse kinematics equation of robot. Using reverse kinematics and coordinates of the object, we can move our end effector of robot to the desired position in the workspace of robot.

Procedure of finding coordinates of centroid of the object placed on conveyor :
  1. First we find ROI (Region Of Interest). This is to focus on the only part we need from image.
  2. Then we find the gray scale image of the same, so that we can get better output from threshold.
  3. Now we find the threshold to differentiate between object and conveyor belt.
  4. We outline the object obtained from threshold and find centroid using formula.
