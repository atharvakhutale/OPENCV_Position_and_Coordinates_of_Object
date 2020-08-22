# opencv_objectdetection
# Draw border and find centroid of object moving on conveyor

import imutils
import cv2
import numpy as np
import urllib.request as urllib2
import time

SOURCE_IMAGE="IMG_1.JPG"
baseURL = "https://api.thingspeak.com/update?api_key=MM9SWBLL3YI3BCVT&"
OBJECT_DETECT=0
OBJECT_TYPE=0
OLD_VERTICES=[[[0,0],[0,0]],[[0,0],[0,0]],[[0,0],[0,0]],[[0,0],[0,0]]]

# aio=Client(ADAFRUIT_IO_USERNAME,ADAFRUIT_IO_KEY)

# load the input IMAGE and show its dimensions, keeping in mind that
# IMAGEs are represented as a multi-dimensional NumPy array with
# shape no. rows (height) x no. columns (width) x no. channels (depth)

cap = cv2.VideoCapture(0)

# Check if the webcam is opened correctly
if not cap.isOpened():
    raise IOError("Cannot open webcam")

while True:
    ret, frame = cap.read()
    # IMAGE = cv2.resize(frame, None, fx=0.5, fy=0.5, interpolation=cv2.INTER_AREA)
    IMAGE=cv2.imread(SOURCE_IMAGE)


    cv2.imshow("IMAGE", IMAGE)
    c = cv2.waitKey(1)
    if c == 27:
        break

    (h, w, d) = IMAGE.shape
    print("width={}, height={}, depth={}".format(w, h, d))
    
    # display the IMAGE to our screen -- we will need to click the window
    # open by OpenCV and press a key on our keyboard to continue execution
    cv2.imshow("IMAGE", IMAGE)
    c = cv2.waitKey(1)
    if c == 27:
        break

    RESIZED_IMG = imutils.resize(IMAGE, height=100)
    # cv2.imshow("Imutils Resize", resized)
    # cv2.waitKey(0)

    CROPPED_IMG=RESIZED_IMG[40:85, 0:120]
    cv2.imshow("ROI",CROPPED_IMG)
    c = cv2.waitKey(1)
    if c == 27:
        break

    GRAY_IMAGE = cv2.cvtColor(CROPPED_IMG, cv2.COLOR_BGR2GRAY)
    cv2.imshow("GRAY",GRAY_IMAGE)
    c = cv2.waitKey(1)
    if c == 27:
        break

    # EDGED_IMAGE = cv2.Canny(GRAY_IMAGE, 30, 150)
    # cv2.imshow("Edged", EDGED_IMAGE)
    # cv2.waitKey(0)

    THRESHOLDED_IMAGE = cv2.threshold(GRAY_IMAGE, 150, 200, cv2.THRESH_BINARY)[1]
    # print(np.sum(THRESHOLDED_IMAGE))

    cv2.imshow("Thresh", THRESHOLDED_IMAGE)
    c = cv2.waitKey(1)
    if c == 27:
        break

    CONTOURS=cv2.findContours(THRESHOLDED_IMAGE.copy(),cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)
    CONTOURS=imutils.grab_contours(CONTOURS)


    for c in CONTOURS:
        # compute the center of the contour
        if cv2.contourArea(c)>350:
            OBJECT_DETECT=1
            peri = cv2.arcLength(c, True)
            VERTICES = cv2.approxPolyDP(c, 0.04 * peri, True)
            
            # print(VERTICES)
            if ((VERTICES!=OLD_VERTICES).all()):
                OLD_VERTICES=VERTICES
                (x, y, w, h) = cv2.boundingRect(VERTICES)
                ar = w / float(h)
                # a square will have an aspect ratio that is approximately
                # equal to one, otherwise, the shape is a rectangle
                if (ar >= 0.99 and ar <= 1.01):
                    shape = "square"
                    OBJECT_TYPE = 0
                else:
                    shape="rectangle"
                    OBJECT_TYPE=1

                print(shape)
                M = cv2.moments(c)
                cX = int(M["m10"] / M["m00"])
                cY = int(M["m01"] / M["m00"])
                # aio.send("centx",cX)
                # aio.send("centy",cY)

                f = urllib2.urlopen(baseURL+"field3=("+str(cX)+","+str(cY)+")")
                f.read()
                f.close()
                # time.sleep(15)

                # f = urllib2.urlopen(baseURL+"field4="+shape)
                # f.read()
                # f.close()

                # f = urllib2.urlopen(baseURL+"field3="+str(OBJECT_DETECT))
                # f.read()
                # f.close()
                # aio.send("obj-detect",OBJECT_DETECT)
                # aio.send("obj-type",OBJECT_TYPE)
                # draw the contour and center of the shape on the image
                cv2.drawContours(CROPPED_IMG, [c], -1, (0, 255, 0), 1)
                cv2.circle(CROPPED_IMG, (cX, cY), 3, (255, 0, 0), -1)
                cv2.putText(CROPPED_IMG, shape, (cX - 20, cY - 20),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 2)
                # show the image
                cv2.imshow("Image", CROPPED_IMG)
                c = cv2.waitKey(1)
                if c == 27:
                    break


        c = cv2.waitKey(1)
        if c == 27:
            break
