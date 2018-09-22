# Unscented Kalman Filter Project Starter Code
Self-Driving Car Engineer Nanodegree Program

In this project utilize an Unscented Kalman Filter to estimate the state of a moving object of interest with noisy lidar and radar measurements. Passing the project requires obtaining RMSE values that are lower that the tolerance outlined in the project rubric. 

This project involves the Term 2 Simulator which can be downloaded [here](https://github.com/udacity/self-driving-car-sim/releases)

This repository includes two files that can be used to set up and intall [uWebSocketIO](https://github.com/uWebSockets/uWebSockets) for either Linux or Mac systems. For windows you can use either Docker, VMware, or even [Windows 10 Bash on Ubuntu](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/) to install uWebSocketIO. Please see [this concept in the classroom](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77) for the required version and installation scripts.

Once the install for uWebSocketIO is complete, the main program can be built and ran by doing the following from the project top directory.

1. mkdir build
2. cd build
3. cmake ..
4. make
5. ./UnscentedKF

## State vector and model

For this project, a Constant Turn Rate and Velocity (CTRV) model is assumed while building the state vector. The state vector contains following components:

1. Position of the object in X axis (px)
2. Position of the object in Y axis (py)
3. Speed, or magnitude of velocity (v)
4. Yaw angle (psi)
5. Yaw rate, or rate of change of yaw angle (psidot)

where X and Y axis are relative to the direction in which the self driving car moves, shown below:

![UKF state vector and model](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/ctrv-state-vector.png)

## Unscented Kalman Filter Implementation Algorithm:

Following goals were achieved as a part of implementation:

1. Build the state vector and the state transition matrix. Derive state transition equation. This represents the deterministic part of motion model. Stochastic part of motion is represented by νak (For magnitude of acceleration) and νpsik (For yaw acceleration or the change in yaw rate). The state transition equation then derived is shown below:

![CTRV model state transition equation](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/ctrv-state-transition-equation.png)

where the noise is modelled by assuming Gaussian noise with zero mean and standard deviation. Hence, standard deviations are σa (For magnitude of acceleration) and σyaw (For yaw acceleration). This is shown below:

![CTRV process noise modelled as Gaussian](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/ctrv-noise-modelling.png)

2. LIDAR measures the distance between self driving car and an object in X and Y axis. Hence, the measurement function for LASER updates, given by H_laser, is a linear transform.

3. RADAR measures the radial distance, the bearing (or angle of orientation w.r.t car) and the radial velocity. This is represented below:

![RADAR measurement](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/radar_measurement.png)

Hence, the measurement function for RADAR updates, given by H_radar, is a non-linear transform.

4. Now that the state transition and measurement functions are derived, Kalman filter is used to estimate the path of moving object. Upon receiving a measurement for timestamp k+1, following processes are triggered:

  a. Kalman filter Predict: To use the state vector at timestamp k (Xk) and **predict** the state vector at timestamp k (Xk+1). This is the updated belief after motion.
  b. Use the measurement and update the belief using Kalman filter **update** once measurement is received.
  
5. Kalman filter predict step is same for LASER and RADAR measurements. This step involves use of Unscented Kalman Filter algorithm to predict the mean and covariance for the next step. Given the belief of state and covariance matrix at state k, Unscented Kalman Filter algorithm consists of following steps:

  a. Generate sigma points: In this step, 2n + 1 sigma points are generated, where n is the number of states in state vector. Here, one sigma point is the mean of state while rest all are chosen on the uncertainty ellipse of the Gaussian distribution.
  b. Predict sigma points: The sigma points generated in step a are then passed through the process model and to predict the sigma points for the state k+1.
  c. Predict the mean and covariance: Predicted sigma points from step b are then used to calculate the predicted mean and covariance for the state k+1.

6. When a LASER measurement at step k+1 is received, use vanilla Kalman filter (since the measurement function is linear) equations to update the predicted belief in step 5.

7. When a RADAR measurement at step k+1 is received, again use Unscented Kalman filter (since the measurement function is non-linear) equations to update the predicted belief in step 5.

8. Take a note of Normalized Innovation Squared (NIS) at each step for both LASER and RADAR. Plot the NIS distribution and fine tune the process noise parameters (σa and σyaw) to attain consistency in the system. Equation for calculating NIS is given below:

![NIS equation](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/nis-equation.png)

9. Calculate the root mean squared error (RMSE) after Kalman filter update at each time step. This is given by:

![RMSE equation](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/rmse.png)

If you'd like to generate your own radar and lidar data, see the
[utilities repo](https://github.com/udacity/CarND-Mercedes-SF-Utilities) for
Matlab scripts that can generate additional data.

## Goals of the project

1. Implement Kalman filter algorithm in C++

2. Build the project and run the executable on dataset of LASER and RADAR measurements returned by [Udacity simulator](https://github.com/udacity/self-driving-car-sim/releases).

3. Calculate and capture the Normalized Innovation Squared (NIS) for LASER and RADAR. Plot the values and check for consistency. If not consistent, tune the process noise parameters σa and σyaw. The final tuned values for σa was 2 m/s2 and σyaw was 2 rad/s2. This resulted in NIS distributions shown below:

![LASER NIS distribution](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/ukf-nis-laser-dataset-1.png)

![RADAR NIS distribution](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/ukf-nis-radar-dataset-1.png)

4. Take a note of RMSE values at the last time step of dataset. Minimize the RMSE to bring it in the range of RMSE <= [.09, .10, 0.40, 0.30] for px, py, vx and vy respectively. RMSE values achieved after fine minimization are shown below:

**Run on dataset 1**

![RMSE on dataset I](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/ukf-rmse-dataset-1.png)




**Run on dataset 2**

![RMSE on dataset II](https://raw.githubusercontent.com/sohonisaurabh/CarND-Unscented-Kalman-Filter/master/image-resources/ukf-rmse-dataset-2.png)


## Steps for building the project

### Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
 * Linux and Mac OS, you can also skip to installation of uWebSockets as it installs it as a dependency.
 
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
  * Linux and Mac OS, you can also skip to installation of uWebSockets as it installs it as a dependency.
  
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
  * Linux and Mac OS, you can also skip to installation of uWebSockets as it installs it as a dependency.
  
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`. This will install cmake, make gcc/g++ too.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x.

* Simulator. You can download these from the [Udacity simulator releases tab](https://github.com/udacity/self-driving-car-sim/releases).

### Running the project in Ubuntu

1. Execute every step from install-ubuntu.sh. This will install gcc, g++, cmake, make and uWebsocketIO API.

2. Build project
  a. mkdir build && cd build
  b. cmake ..
  c. make
  d. ./UnscentedKF

3. Run the Udacity simulator and test the implementation on dataset 1 and 2.
