# DeepStream Notes and Resources

## 1. What DeepStream is doing behind the scenes
When running the `source4_1080p_dec_infer-resnet_tracker_sgie_tiled_display_int8.txt` configuration, DeepStream is demonstrating its real-time processing capabilities:

1. **Simulating 4 Cameras:** It uses `num-sources=4` to take a single video file and play it 4 times simultaneously, simulating a system with 4 security cameras.
2. **Running AI Object Detection:** It passes every frame through a Primary AI model (ResNet) to actively search for objects like Cars, People, Bicycles, and Road signs.
3. **Tracking Objects:** It gives every detected object a unique ID number and tracks them as they move across the screen (e.g., "Car #5").
4. **Secondary AI Classification:** Once it finds a car, it passes the cropped image of that car to a Secondary AI model to figure out extra details like Vehicle Type (Sedan, SUV) and Vehicle Make (Honda, Toyota).
5. **Tiled Display (Combining the Videos):** It stitches all 4 video streams with bounding boxes and text into a single 2x2 tiled window and displays it.

## 2. Building a Custom Project: Pothole Detection Roadmap
To build a custom project like pothole detection, you build the model outside of DeepStream and then plug it in:

* **Step 1: Gather and Label Data**
  Find hundreds of images/videos of potholes (e.g., on Kaggle or Roboflow) and use a tool like CVAT or Roboflow to draw bounding boxes and label them "pothole".
* **Step 2: Train the Custom AI Model**
  Use NVIDIA TAO Toolkit (highly recommended for DeepStream compatibility) or YOLOv8 to train an Object Detection model on your pothole dataset.
* **Step 3: Export the Model for DeepStream**
  Convert your trained model into an optimized format like ONNX (`.onnx`) or TensorRT Engine (`.engine`).
* **Step 4: Create the DeepStream Configuration**
  Copy an existing `deepstream-app` config file. Update the `[primary-gie]` section to point to your new Pothole ONNX/Engine file, and update the labels text file to list "pothole".
* **Step 5: Run Your Custom Application**
  Run `deepstream-app -c your_pothole_config.txt` to see real-time pothole detection on your videos.

## 3. Free Resources to Learn DeepStream

* **Free NVIDIA DLI Course (Best for Beginners):**
  Search the NVIDIA Deep Learning Institute (DLI) for the free course: *"Getting Started with DeepStream for Video Analytics on Jetson Nano"*. It teaches you how to build a pipeline from scratch using Python and C++.
* **The Official DeepStream Python Bindings (GitHub):**
  Search for *"DeepStream Python Apps GitHub"* (repository: `NVIDIA-AI-IOT/deepstream_python_apps`). It contains sample applications from "hello world" to multi-camera YOLO detection.
* **NVIDIA DeepStream Developer Documentation:**
  Check out the official Developer Guide: [https://docs.nvidia.com/metropolis/deepstream/dev-guide/](https://docs.nvidia.com/metropolis/deepstream/dev-guide/). Read the "DeepStream Reference Application - deepstream-app" section to understand the config file parameters.

## 4. Commands for the videos you ran

Here are the exact commands you can copy and paste to run the different videos you have tested:

**1. Run the Office Video (`sample_office.mp4`):**
```bash
cd /opt/nvidia/deepstream/deepstream-7.1/samples/configs/deepstream-app/
cp source4_1080p_dec_infer-resnet_tracker_sgie_tiled_display_int8.txt source_office_video.txt
sed -i 's|uri=file://../../streams/sample_1080p_h264.mp4|uri=file://../../streams/sample_office.mp4|g' source_office_video.txt
deepstream-app -c source_office_video.txt
```

**2. Run the Walking Video (`sample_walk.mov`):**
```bash
cd /opt/nvidia/deepstream/deepstream-7.1/samples/configs/deepstream-app/
cp source4_1080p_dec_infer-resnet_tracker_sgie_tiled_display_int8.txt source_walk_video.txt
sed -i 's|uri=file://../../streams/sample_1080p_h264.mp4|uri=file://../../streams/sample_walk.mov|g' source_walk_video.txt
deepstream-app -c source_walk_video.txt
```

**3. Run the Bike Video (`sample_ride_bike.mov`):**
```bash
cd /opt/nvidia/deepstream/deepstream-7.1/samples/configs/deepstream-app/
cp source4_1080p_dec_infer-resnet_tracker_sgie_tiled_display_int8.txt source_sample_ride_bike.txt
sed -i 's|uri=file://../../streams/sample_1080p_h264.mp4|uri=file://../../streams/sample_ride_bike.mov|g' source_sample_ride_bike.txt
deepstream-app -c source_sample_ride_bike.txt
```

**4. Run the Fisheye Video (`fisheye_dist.mp4`):**
```bash
cd /opt/nvidia/deepstream/deepstream-7.1/samples/configs/deepstream-app/
cp source4_1080p_dec_infer-resnet_tracker_sgie_tiled_display_int8.txt source_fisheye_dist.txt
sed -i 's|uri=file://../../streams/sample_1080p_h264.mp4|uri=file://../../streams/fisheye_dist.mp4|g' source_fisheye_dist.txt
deepstream-app -c source_fisheye_dist.txt
```
