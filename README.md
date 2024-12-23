Camera array-based head direction detection method

Summarized by Yuheng F. (01/22/24)

Main idea: Use an object of known shape/size fixed on bat’s head, and multiple cameras that monitor the object from different angles. By combining the views of this object under different cameras, we can decode the orientation of the object relative to the camera system.

Components:
Target object: flight cap with 3 beads on top
flight cap: flight cap should be tall enough to be able to fully close and form a consistent rigid body with the bat’s head. Variant relative position will cause failure in detecting accurate head direction. (But still could be used to train model to detect markers. These are two separate task/steps)
beads arrangement: 3 beads arranged in L shape were the empirical valid choice. Note they should be arranged maximally non symmetrical (the two edges of L should be of different length)(recommended method: put two beads at two farthest edge of the rectangular part of cap top; then place the third bead so that the connecting line between it and one of the marker is forming 90 degree with the initial line of two markers; The distance of the third marker to the initial line should be 3:4:5 (calculated by: c-b=b-a and a^2+b^2=c^2).
Camera array: Although the more the better, a minimal number is two cameras on top of the object pointing down slightly at opposite angles toward the object. A third camera could be placed in the middle of these two cameras pointing straight down. 
Note 1: in the whole camera system, at any moment, at least two cameras should be able to see all three beads (these two cameras could be different across frames).
Note 2: camera exposure time should be set as around 5000 us (frame rate could be 30hz). Excessive long exposure cause severe motion blurriness of markers when the bat’s moving head and licking smoothie from spoon, which is hard to label the marker and detect the position. 
Marker(s) (board) for camera calibration: 
Two types of marker(s) could be combined for calibration 
A board with multiple calibration markers is usually held under cameras and manually rotated to sufficiently cover potential angles and distances to each camera before experiment, in order to calibrate the whole camera system against drifting. See more details of this step in the Procedure section.
Another marker is attached to the platform where the device is placed on, so that it’s viewed by cameras during the whole experiment. This marker could be used to calibrate cameras within frame in the recorded video (and thus okay if accidentally displaced a bit during recording)
For this purpose, generate a charuco board on this website. Make sure the marker patterns in each board don't overlap (if they might appear simultaneously under cameras), which could be achieved by setting the Start ID in the option when generating the board. And during analysis, use this package to calibrate cameras.
Aluminum frames: Essentially constructed using t-slots (20mm is the thin one that the sniffing cage is mainly built with, but thicker is desired based on weight of load), hinges and screws. Ordered from misumi website. Important note: need to determine all the length and size of the t-slot bars on ordering and items will be delivered after cutting. Stick to metric measurements. 
Reference object: Some landmarks that naturally exist on site (eg, microphones on the side) could be used in downstream analysis to transform the orientation relative to camera into relative to desired reference system (for example, let x axis align to some edge)

Procedure:
Pre-experiment calibration: Calibrate with the board of markers once a few days is enough to cancel the slow drifting of the system. During the calibration process, make sure that the board is not constantly moving, and is instead static for a few frames at each transient position to avoid motion blurriness.
Record with camera array: Another calibration marker should be placed next to the animal. Infrared light could be used if wanted, to emphasize reflected light from beads.

Analysis:
Camera calibration: this is done with Aniposelib python package which iteratively does the calibration and triangulation process based on calibration videos taken by multiple cameras where a charuco (or checker alternatively) board is placed and moved. Each marker in charuco board provides a complete projection information. The error of calibration is given in the calibration.toml file where “error” is indicated in the unit of pixel (so 0.24 is pretty decent calib error).
In the demo of aniposelib, in the end a 3d hand joint map is shown but that’s just a preloaded 2d coordinates back projected to 3d. Object detection is not carried out in this package.
Marker detection: Basic idea is using the pretrained marker detection model in lab (called ir_marker_detector_yolov8; trained in other scenarios, like flight room or Kevin’s cage) and fine tune it with a smaller training set created based on my own application scenario (like restrained bat experiment).
Image set preparation: go over the video (frames) and pick out some frames that are tough for computer to detect markers. A total number of 50 frames would be good for training set. And validation a quarter of training set. Important: only put in training set frames/images that you’re gonna really label the markers out of. Don’t intend to leave empty frames (with no labeling).
Manual labeling: create training and validation set in labeling website using these frames. First in each set, run “action - automatic detection” to first pass label the markers using the pre-trained model to save time. Then go frame by frame to square out marker (best square; infer if partial hidden). (short cut: N to mark, and F to move to next frame)

Object orientation decoding: calculate the object orientation relative to the camera system in Python
Transformed into desired reference system: Mouse click some reference objects visible under cameras and then specific Python script could transform the orientation
