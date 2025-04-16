
# Integrating Webots and FEAGI
By Nathaniel Balkaran, Sophia Thompson, and Shizhen Yang

For our capstone project at the University of Pittsburgh, we set out to integrate [Webots](https://cyberbotics.com/), a profesional robot simulator, and [FEAGI](https://www.feagi.org/), an open-source platform for biologically-inspired artificial general intelligence. Our goal was to create a bi-directional interface, enabling a virtual robot to be controlled by an evolving AI brain.

To achieve this, we developed a custom Webots controller that acts as a communication bridge between FEAGI and the simulation environment. Sensor data is extracted from the robot, then formated so FEAGI could process and transmit to the brain. After processing, the brains actuator commands are recieved and used it to manipulate the robot.

## Exploring Webots

Webots provides a rich environment to simulate a wide variety of robots. Its real-time sensor and actuator interface made it a perfect match for experimenting with brain-controlled behavior. Before diving into integration, we explored the Webots system independently to get comfortable with its environment, learning how to extract and print sensor data and articulate the robot by feeding motors commands.
![Screenshot-2025-04-10-112843.png](Screenshot-2025-04-10-112843.png "pr2 robot")
*Webots sample world with robot called pr2 and a table with objects for it to pick up*

## Development 

The first thing we had to do was get familiar with Webots. With the help from the [webots reference manual](https://cyberbotics.com/doc/reference/index), we learned how to interact, obtain names, and types of all devices on a robot (such as cameras, distance sensors, motors, etc). Next, our goal was to learn how to run the controller from our computers terminal instead of executing locally in the webots program. This makes the process of running controllers more versitile, letting us switch between different controller files and robots faster. From our initial exploration of moving the pioneer2 and pr2 robots, we realized that Webots does not have a native servo motor device, but instead uses a rotational motor set to either velocity or position control mode. 


We were then provided with a FEAGI connector template file that created a pipeline and gave an example of how to recieve and send data to FEAGI. To be able to exchange FEAGI data, the brain needs an initialization of a FEAGI supported device. This initialized device is held in the capabilities.json file. It holds data such as the name, physical limits, or other settings for each device. We created another file that generates this json file and saturates it with the robots device data we read every time the controller is executed. Webots has devices that FEAGI does not support, like GPS, brakes, speakers, and more. These devices are currently not being read by our controller, but it would be very easy to add this functionality when FEAGI has these devices supported.


When speaking to FEAGI, data needs to be formatted in a specific way. For example, a problem that we faced was processing the data from camera or lidar sensors in webots. Webot's cameras send data as a string of bits in RGBA, and FEAGI needs a 2D array holding R,G, and B, arrays. We needed to convert the string to properly formatted arrays, and then delete the A channel. 


## Genome 
Lastly, we each designed a custom genome (available [here](https://github.com/feagi/feagi/tree/staging/community_genomes)) that allow for more complex control patterns. The salamander and pioneer2 genomes both move the robot in a straight line unless there is a wall in front of them, in which case they turn to avoid it. The Tiago genome allows it to move forward and backward.

(insert genome video)

## Challenges
We faced a few challenges while completing this project, some of which we have already mentioned above. Others are listed below

- **Outdated Webots Documentation**  
  Some examples were broken or referenced deprecated APIs.

- **FEAGI Docker Build Issues**  
  The Docker images required manual fixes for compatibility with newer systems.

- **Continuous Camera Output Lag**  
  Streaming camera frames to FEAGI introduced noticeable latency and performance issues.

## Final Thoughts
Despite challenges, we successfully achieved integration between Webots and FEAGI. Anyone curious or inspired to explore what a biologically-inspired AI-powered robot can do, now has an easy way to get started. This sets the stage for future research in learning-based behaviors, prosthetic simulations, and neuro-robotics.