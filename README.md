# Combined_code_groom
## Import
- Standard Python libs: `sys`, `os`, `time`, `copy`, `math`, `threading`, `statistics`, `csv`
- ROS packages: `rospy`, `std_msgs`, `sensor_msgs`, `navigation.msg`, `traversal.msg`
- Image processing: `cv2`, `imutils`
- Depth & vision: `pyrealsense2`, `ultralytics`, `open3d`, `cv_bridge`
- Others: `numpy`, `collections`

## Subscribers
- `/zed2i/zed_node/rgb/image_rect_color` : RGB images (ZED/RealSense)
- `/zed2i/zed_node/depth/depth_registered` : Depth images
- `/zed2i/zed_node/imu/data` : IMU orientation
- `/gps_coordinates` : GPS (NavSatFix)
- `state` : Boolean for autonomous/manual mode
- `enc_arm` : Encoder angle for Realsense head movement

## Publishers
- `motion` : traversal.msg.WheelRpm (drive/steering commands)
- `gps_bool` : Competition/event-driven GPS trigger
- `rot` : Integer, steering mode selector (rotate in place, straight, etc.)
- `stm_write` : MultiArray for Realsense base rotation

## Attributes:
- `state_callback`: Tracks current state (autonomous or manual)
- `color_callback`, `depth_callback`: Update current RGB and depth images
- `yaw_callback`: Decodes current yaw angle from IMU data
- `enc_callback`: Stores current encoder feedback (for Realsense pan/tilt)
- `gps_callback`: Updates present latitude/longitude
- `get_box`, `cone_model`: Apply YOLOv8 models to detect arrows and cones in RGB images, annotate results
- `arrowdetectmorethan3`,`arrowdetectlessthan3`: Uses YOLO detection & OpenCV template matching to identify arrows, calculate direction and confidence
- `move_straight`: Controls forward robot motion, with velocity logic based on distance to arrow/cone
- `rotate`, `rot_in_place`: Handles robot turning behavior (regular and in-place)
- `search`: Rotates Realsense to actively search for manually lost arrows, collect multiple view angles
- `process_dict`: Decides best rotation based on detection angle dictionary
- `main`, `run`: Robot state machine loop for perception→decision→motion

## Control Flow
1. Initialisation
   - Instantiates `ZedDepth` class, registers node & parameters
   - Subscribes/Publishes to all relevant ROS topics for vision, control, and feedback
   - Loads arrow/cone detection YOLO weights and template images for direction detection

2. Arrow detection
   - Receives latest color and depth frames
   - Runs YOLO and, if needed, template matching for robust arrow/cone identification
   - Computes distance using depth image and computes confidence for detection

3. Decision making
   - Tracks arrow passage count; triggers new navigation strategy or mode switch after completion
   - Calculates correct robot movement: forward, rotate, or in-place, depending on what is detected (and fail-safes if nothing is seen)
   - Searches actively in a “burst/scan” manner if detection is persistently lost

4. Execution
   - Publishes `WheelRpm` messages for drive/steer output
   - Records GPS to file after competition/end of run
   - Writes annotated images and detection overlays for debugging/validation



