\page calibration Calibration

# Package Overview
# Camera Target Based Intrinsics Calibration 
This package refines an initial estimate for camera intrinsics and distortion values given a bag file containing images of a calibration target and a calibration target configuration.
Several camera distortion models (fov, rad, radtan) are supported for calibration and various scripts are provided to view the quality of the input target detections and the calibration results. <br />

For generating an initial estimate of camera intrinsics and distortion values, see https://github.com/nasa/astrobee/tree/master/scripts/calibrate.

## Example Usage
The following gives an overview of refining calibration parameters given an initial estimate and a bag file containing target detections. 
For more information on each tool or script and its options, see [Usage Instructions](#usage-instructions). 
### Generate target detections from bagfiles
`rosrun calibration save_images_with_target_detections -d ~/bag_files -o target_detections -t ~/target.yaml` <br /> 
Here `~/bag_files` contains a directory of bagfiles containing target detection images and target.yaml is the desired target yaml file found in https://github.com/nasa/astrobee/tree/master/scripts/calibrate/config.
### View target detection coverage in image space
`rosrun calibration view_all_detections.py -d target_detections`<br /> 

![Detection Image showing detected target points in image space in green.](/doc/images/calibration/detection_image.jpg)

Ideally the target detections span the entire image. If only the middle of the image contains detections or if parts of the image have no detections, this may lead to an underconstrained calibration problem.  Ensure good target coverage before attempting to calibrate a camera. <br />
### Calibrate
#### Calibration Parameters 
In addition to the script parameters, the camera_target_based_intrinsics_calibrator.config contains most of the configuration options for camera target based intrinsics calibration, found here: https://github.com/nasa/astrobee/tree/master/astrobee/config/tools/camera_target_based_intrinsics_calibrator.config.  The distortion model, camera name, parameters for the reprojection pose estimator handling target pose estimation, visualization, and other general parameters can be selected. <br />
The parameters to calibrate can also be selected.  Depending on the distortion model used, different parameters should be calibrated concurrently to avoid degeneracies and underconstrained calibration.  For example, calibrating the camera principal points and target poses at the same time should be avoided.  Calibration may be repeated while incrementally updating solved parameters and toggling different concurrent parameter sets to solve for until all parameters are adequately calibrated.  
#### Run Calibration
`rosrun calibration calibrate_intrinsics_and_save_results.py target_detections ~/astrobee/src/astrobee/config config/robots/bumble.config -u -p -i target_detections -d fov` <br />
#### Calibration Output
The calibrated results are saved in the calibrated_params.txt file while more verbose output is saved to the calibration_output.txt file.  
#### Judging Calibration Results
Rely primarily on the covariances for each calibrated parameter reported at the end of the calibration_output.txt file. <br />
##### Reprojection Image
The output calibrated_reprojection_from_all_targets_absolute_image.png displays reprojection errors after the calibration procedure has completed and can give a general sense of the calibration accuracy, although it does not discern between accurate calibration and overfitting, which is why the covariances mentioned previously should be the primary indicator of calibration accuracy.<br />
Here reprojection errors are colored by the norm of the error on a continuous color spectrum, with dark blue indicated small/zero error and red indicating large error.  
![Reprojection image showing error norms, with blue indicating small error and red indicating large error.](/doc/images/calibration/calibrated_reprojection_from_all_targets_absolute_image.png)

##### Error Histogram
The generated error histogram (which displays the overall norm, x, and y errors on seperate plots) can be helpful to view overall error during calibration and to see if any bias exists in the errors.  For example, if the x or y error histograms are not zero centered, this can indicate an error in the calibrated principal points. <br />
![Norm errors](/doc/images/calibration/norm_errors.png) 
![X errors](/doc/images/calibration/x_errors.png) 
![Y errors](/doc/images/calibration/y_errors.png) 

This is generated by passing the -p option to the calibration script or by running the make_error_histograms.py script on the output errors.txt file after calibration has completed.

##### Undistorted Images
The undistorted images generated by passing -u to the calibration script or by seperately running the create_undistorted_images script give an additional indicator of calibration accuracy.  Undistorted calibration targets should have straight boundary edges and straight lines within the target.  Curved lines can indicate an error in the calibration distortion parameters. <br />

Distorted Image:

![Distorted image.](/doc/images/calibration/distorted.png) 

Undistorted Image (FOV distortion):

![FOV undistorted image.](/doc/images/calibration/fov_undistorted.png) 

The black dots in the undistorted image are a result of the FOV undistortion procedure, while the Rad and RadTan uses interpolation to fill these. 

# Usage Instructions
For each script and tool, run `rosrun calibration script_or_tool_name -h` for further details and
usage instructions.

# Tools
## create_undistorted_images 
Generates undistorted images from a set of distorted images and provided camera calibration parameters.

## run_camera_target_based_intrinsics_calibrator
Runs the intrinsics calibrator using a set of target detection files and initial estimates
for the camera intrinsics and distortion values.  Support various distortion models.

# Scripts
## calibrate_intrinsics_and_save_results.py  
Runs camera intrinsic calibration using provided target detections and a config file with camera
parameters (including initial estimate for camera intrinsics and distortion values).

## copy_calibration_params_to_config.py      
Helper script that copies calibration parameters from the output of the 
calibration pipeline and writes these to the camera config file.

## get_bags_with_topic.py    
Helper script that generates a list of bag files in a directory
with the provided topic. 

## make_error_histograms.py
Generates a histogram of errors using the output errors from 
the calibration pipeline.

## save_images_with_target_detections.py
Generates target detection files for use with the calibration 
pipeline from a set of bagfiles containing images of target detections. 

## view_all_detections.py
Generates an image containing all images space detections of a target
for a set of target detection files.