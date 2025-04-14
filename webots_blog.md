# Integrating Webots and FEAGI
By Nathaniel Balkaran, Sophia Thompson, and Shizhen Yang

For our capstone project at the University of Pittsburgh, our goal was to integrate [Webots](https://cyberbotics.com/) with [FEAGI](https://www.feagi.org/). Webots is an application used in industry, research, and education to model, simulate, and program robots. To do this, we created a Webots controller that acts as a bridge between FEAGI and Webots. First we extract sensor data from the robot, then convert it into a format FEAGI could process and send it to the brain. Next, we pull the actuator data from FEAGI's brain, and used it to control the robot.

## Development 

The first thing we had to do was get familiar with Webots, as none of us had any experience with it. With the help from [webots reference manual](https://cyberbotics.com/doc/reference/index), we learned how to get the names and types of all deviced on a robot (such as cameras, distance sensors, motors, etc), and get continous data from the sensors. We also learned how to use an external controller (i.e. a controller not within Webots) on robots. We also created simple controllers to move the pioneer2 and pr2 robots. Through this, we realized that Webots does not have servo motors, and instead uses a rotational motor in combination with a position sensor. All of this was very useful as we moved into developing our controller.

We were provided with a FEAGI connector template that already contained code to send data to FEAGI, which allowed us to focus on extracting the data from Webots and then using data sent from FEAGI to control the robot in Webots. 

Before we could send any data to FEAGI, we had to decide what type of FEAGI device each device in Webots is. For some devices this was obvious; for example Webots distance sensors are FEAGI proximity sensors. But in other cases it was less clear. As mentioned above, rotational motors in Webots can also be used as servos, so we had to determine which way one was being used. To do this, we used the maximum and minimum positions of the motors. These fields will both be a non-zero number if the motor is acting as a servo. Webots also has devices that don't have any equivalent in FEAGI, like GPS or receivers and emitters. These devices are not compatible with our controller, but it would be very easy to update the controller to support them in the future.

Once the Webots devices are sorted into lists of their corresponding FEAGI devices by the controller, we use them to generate the capabilities of the robot. We used the FEAGI Configurator in Godot to create some example capabilities files, then made sure that our automatic generator makes a correctly formatted file. The capabilities generator is run automatically every time the controller is run.

Any data that is going to be sent to FEAGI needs to be formatted for FEAGI to understand.


- formatting data 
  - touch sensors
  - converting camera/lidar data to send to FEAGI
- creating our own genomes
