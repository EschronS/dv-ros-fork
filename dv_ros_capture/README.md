# Configuration for running on Raspberry Pi 5 in Docker

This repo contains my changes to the dv_ros_capture node for recording
event data bags on a Raspberry Pi 5 compute module and a DVXplorer Micro.

## Goal of my changes

Prevent or minimize event drops and IMU timestamp inconsistencies when
recording event data ROS bags on a Raspberry Pi 5 compute module.

## Measures taken

- increased contrast threshold
- increased queue sizes (here and in the dependency dv-processing,
  see: [dv-ros-docker](https://github.com/EschronS/dv-ros-docker-rp5))
- publishing at 100 Hz instead of 1000 Hz to reduce ROS overhead
- handle duplicate IMU timestamps by incrementing the timestamp by the
  inverse IMU frequency (the Micro has no proper IMU clock)
- implemented a frequency low-pass filter, rejecting events above a local
  frequency at any pixel (using dv-processing)

## Notes

- the noise filter provided by dv-processing was tested but was very
  expensive, so it is not included in these changes
- profiling of the running node can be done with
  `sudo perf top -p $(pgrep capture_node)`

## Other relevant repos

- my [dv-ros-docker](https://github.com/EschronS/dv-ros-docker-rp5)
  setup including the patched dv-processing
- my ROS 2 driver alternative:
  [dv_processing_driver](https://github.com/EschronS/dv_processing_driver)
- current official dv-ros on [gitlab](https://gitlab.com/inivation/dv/dv-ros)

# DV ROS Capture

This project contains a capture node that can is intended for live camera and aedat4 recording data publishing in ROS.

The node features:

- Publishing data from a live camera connected over USB.
- Playback data from an aedat4 file and publish it into ROS topics.
- Load and save calibration files using DV format.

# Running the node

The node will try to detect and open a camera connected through USB if no settings are provided.

Aedat4 playback will be performed if a path to an aedat4 file is set as the parameter "aedat4FilePath".

It is also possible to specify an exact camera to open using the "cameraName" parameter. Camera name consists of camera
model and a serial number concatenated by a '\_' symbol, e.g. "DVXplorer_DXA000001". In live mode, this will open only
the specified camera, in aedat4 playback it will filter data streams that were recorded from the specified camera only
that are available in the file.

Please see `config/settings.yaml` for a full list of available settings.

# Calibration files

Calibration files are stored under `~/.dv_camera/{cameraName}` directory. The node will try to find "active calibration"
file named `~/.dv_camera/{cameraName}/active_calibration.json` and will publish the calibration into `~camera_info`
topic. Active calibration is saved when a `set_camera_info` service is called.

This calibration can be overridden manually using the `cameraCalibrationFilePath` parameter, this parameter will not
replace the "active calibration" file, it only loads calibration from the given path instead.

If no calibration is provided or detected, the capture node will issue a warning message and publish "ideal" calibration
which assumes no distortion, central point as the center of the image pixel space, and focal length equal to the width
of the image.

# Multiple camera synchronization

Multi-camera setups connected with synchronization cables require synchronization signals to synchronize the data
streams. This can be achieved by setting up the launch files on which cameras need to be synchronized. This guarantees
the synchronization to happen at correct time, since all cameras need to be opened prior to sending synchronization
signal. Please refer `launch/synchronization.launch` file for details on how to set up the multi-camera synchronization.

## License

Derived from iniVation's dv-ros and distributed under Apache-2.0. See
[LICENSE](../LICENSE) and [NOTICE](../NOTICE).
