# Model Predictive Controller Project
Self-Driving Car Engineer Nanodegree Program

---

## Model

#### State

The state of the vehicle is stored in 6 variables. The first four variables represent the physical state of the car, meaning the x and y position, the orientation or heading angle, and the velocity. This is the information you need at any point in time to locate the car on a map and determine where the car is headed.

The last two variables are the cross track error and the orientation error. These represent the difference between where the car is and where we want the car to be. These values are calculated from the current position and the desired path.

#### Actuators

There are two main controls for a car, the steering wheel and the pedals (the gas provides a positive acceleration  and the brake a negative one, so we can treat them as doing the same thing just in opposite directions). The model has two variables for determining these controls. One is the steering angle which determines how we turn the wheel and the other is the acceleration which determines which pedal we push and how hard.

We place some constraints on the allowed values for these variables. Cars have a physical limit to how sharp they can turn. Thus we constrain the steering angle to 25 degrees in either direction. We limit the acceleration to be between -1 and 1, signifing full brake or full acceleration.

#### Update Equations

In order to predict the behavior of the car we need to know how the state of the car will change given a starting state and the actuator values. Several of these equations or fairly straightforward. The x and y position will change based on the velocity and the orientation of the car. The velocity of the vehicle will change based on the acceleration. The orientation of the car will change based on how sharply the wheel is turned, the current velocity of the car, and the length of the car.

The trickier update equations are the error values. The cross track error measures how far away the car is from the desired position. This is equal to the previous cross track error plus the change that occured based on the orientation error (if the car is oriented correctly there will be no increase in the cross track error, if it is oriented away from the correct path it will increase, if oriented towards then the cross track error decreases). Similarily, the orientation error will increase or decrease based on the change that the steering angle produces.

## N and dt

When calculating the path the car will take we have to determine how far ahead to calculate. I figured one second was a good length of time to use. Knowing I wanted about one second of total look-ahead time I had to decide how many time steps to use and what the length between each time step would be. Make the duration between time steps too small and you'll need so many timesteps the optimizer won't have time to finish its calculations. Make the time steps too big and you don't account for changes often enough.

In the end I used 12 timesteps of 0.08 seconds duration. Based on the fact that the latency is 0.1 seconds I knew that around that number was a reasonable time to wait between calculations. I played around a bit and found that 12 and 0.08 gave good results.

## Vehicle Coordinate Transform

The simulator provides the desired path of the car via waypoints in the global map coordinate system. From those waypoints we calculate a polynomial and have the car follow that polynomial's path. In order to make things easier, I first transform the waypoints from the map coordinate system to the vehicle coordinate system. That means the vehicle starts at a x,y postion of 0,0 with a 0 degree orientation. Doing it this way allows for easier communication with the simulator.

## Latency

Once I got the controller working I had to deal with the latency. A real car will have a delay between when a command is given and when the wheels physically turn. This is simulated by delaying each command by 100 milliseconds. To account for this, I calculate where the car will be in 0.1 seconds in the future, then use that value for the starting position of the car. I use the same equations that we used before to calculate a future state based on the current state and the actuator values. This allows the model to calculate the right steering angle and acceleration based on where the car will be when those commands actually get executed.