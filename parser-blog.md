# Parsing Gazebo for FEAGI

#### Lukas Finn, Timothy Marshall, Brenden Vaccaro, Benson Beck

During this semester, our team has worked on enabling FEAGI's artificial general intelligence (AGI) to control a wide variety of robotic simulations that originate from Gazebo, a robotics simulation software platform designed for simulating the control of robots and their environments.	We created a parsing program to convert simulation data from Gazebo into a readable format for FEAGI to configure and interpret so that their AGI can successfully control the simulation's robots.

## Using FEAGI With Gazebo

The following diagram demonstrates the desired functionality between Gazebo and FEAGI for successful robot control. This semester, our team focused on the sections in green to develop.

![Visualization of how Gazebo and FEAGI can work together, with a focus on the parsing of Gazebo SDF files](https://github.com/user-attachments/assets/c0d6de78-a5a5-40cd-9d3d-7e7846c37ad5)

Our team was responsible for creating a parsing program that converts Gazebo SDF files into readable JSON output to be used by the FEAGI Configurator, which will then generate necessary data for the Gazebo Controller. Gazebo allows for the use of many different inputs/sensors and outputs/actuators in their simulations, and all simulation data is stored in SDF files. The formatting of the data in these SDF files are not uniform, as projects will have different goals and types of robots. In order for the FEAGI Configurator to understand any given simulation, it needs to be fed a standardized JSON format. We successfully created a parser that adapts to simulations of all kinds and produces a proper JSON configuration tree that the Configurator can use to produce a JSON listing capabilities for the Gazebo Controller.

## Development of the Parser

The Configuration Parser is a python program that uses an XML Element Tree library called lxml to convert a Structured Data File, or SDF, into a uniform JSON configuration file. The program works by using a list of allowed xml elements as well as elements that should be skipped and mapping them onto specific FEAGI device types. 

### First Steps

The first milestone that the team was given was to familiarize ourselves with the formatting of SDF files and test out configurations on the Gazebo Simulator so that we are able to understand how the parsed files should function in the future. 

We have found the three main necessary categories for parsing: sensors, actuators, or links.

Sensors take in data from the environment around them and their interactions (input).
![Chart of sensors/input used by FEAGI](https://github.com/user-attachments/assets/2e4523e6-c742-4e5c-850b-c202400a37b8)

Actuators use the input from the sensor or user (in this case, FEAGI) to complete actions (output).
![Chart of actuators/output used by FEAGI](https://github.com/user-attachments/assets/daafbd05-373c-47ca-a84c-fdf6f48f0621)

Finally, links are the general structure of the machine that demonstrate how the whole robot is held together. 

It is incredibly important to be able to distinguish between all of these different parts of a robot. In order for FEAGI to correctly control the robot, it needs to know exactly what its capabilities are. Even if some sensors or actuators seem similar, they serve different purposes and have different data properties, meaning they cannot be incorrectly labeled for FEAGI to control.

Knowing this, we can set the goal to parse the SDF file to find all these essential robotics parts, correctly categorize them, fill in proper data to FEAGI specs, and correctly restructure new data. This will allow FEAGI to successfully take control of robotics simulations.

### Creating the Program

The next milestone given to the team was to create the configuration parser. In order to complete this milestone, the team needed to first create a list of mappings from xml elements to FEAGI devices, and parse through the SDF file to find data relating to each element. After a mapping was created and the program was able to find the needed elements, the team was tasked with properly nesting the elements in the JSON configuration file to allow for better funcitonality within the FEAGI controller. This process is demonstrated in the diagram below.

![Flowchart demonstrating the parsing process and how it will be used](https://github.com/user-attachments/assets/c3c6893e-59ff-46e2-857c-0b3a2dccd343)

The first step of the program is to convert the SDF file into an XML Element Tree. Due to compatibility issues, we ended up using the LXML package to do this. This step is done so that we can more easily analyze and parse the incoming file. Once we have the XML ETree, our programs begins the parsing process by scanning the entire XML ETree, finding and storing all elements included in the 'allow list' of the ```gazebo_config_template.json``` file.

From there, our program refers to the ```feagi_config_template.json``` file to find all properties of the found devices, and inputting them into a list of dictionaries. This occurs for all found elements until we have a list full of dictionaries, each index holding a device or link.

The next step is nesting the elements according to their 'parent' and 'child' properties. This was the tricky part to figure out, and there were many functions created in order to try to solve this. In the end, we decided on a function that scans through the XML tree, checking for 'parents' and 'children' of the devices and links. Depending on which relationship was found, either the current element had another child added into the list inside the dictionary, or the current element was moved into their parent's list of children. This successfully nested all the devices and links, and now the list was ready for the final step.

Once everything is correctly nested, the list is dumped in JSON format into ```model_config_tree.json```, where it will be used by the FEAGI Configurator on Godot.

### Experience Along the Way

Through the development of the parser, we had learned a lot about parsing and gained valuable experience working with SDF, XML, and JSON files. We also had to overcome challenges along the way. Some challenges faced include: Gazebo SDF files coming in different formats, team members' ability to work on code together, and code clarity. 

To overcome the challenge of SDF files having no single format, we worked on one SDF file at a time, ensuring full compatibility with one format at a time, and slowly expanding the compatibility on other files until multiple incoming formats are accepted, while still producing one uniform output format.

Initally, different team members used different IDE platforms for the project. While our communication was good, something felt missing, and we would still run into issues such as merge errors. To solve this, we decided to all use Visual Studio Code with the 'Live Share' plugin. Besides ensuring we all have the same capabilities and tools accessible to us, we also were able to work collaboratively in real-time, ensuring we do not overwrite each other's work or have to deal with irritating merge errors. This allowed us to test and show our code to each other as we wrote it, getting immediate feedback, allowing us to code more efficiently.

Through our development, we had a slowly growing 'graveyard' of unused functions, new functions missing documentation, and variables with rather generic names (such as 'myList'). This was unacceptable, so we tackled these issues by removing the unused code to give the program a clean feeling. Additionally, we filled in proper documentation for all of the functions that are used and variables were renamed to better demonstrate their purpose and so that users who inspect the code can understand how the parser works.

## Results & Final Notes

We were successfully able to construct the parsing program and convert Gazebo SDF files into a standard JSON-formatted model configuration tree. This is passed to the FEAGI Configurator on Godot, where it generates a JSON listing the capabilities of the robot. FEAGI then uses this file with Gazebo in the FEAGI Controller, where it is able to successfully control the robots and receive the proper input. Shown below is a short video of one robot that we had successfully parsed using its camera sensor.

![Robot successfully using camera sensor](https://github.com/user-attachments/assets/8df4e746-3036-4b9f-bf46-e7ed0563682b)

Here's another example of our work in action - a robot that we parsed is now able to seamlessly be controlled through FEAGI's brain visualizer in Gazebo.

![Robot successfully being controlled through FEAGI's brain visualizer](https://github.com/user-attachments/assets/f24cf90e-7a62-4779-9d0d-5930642299a0)

Our work simplified the process of turning simple robotic models into AI-controlled machines using FEAGI. The parser makes this process incredibly simple and efficient, enabling greater accessibility to users of differing skill levels and purpose. 

Our team had an amazing time working on this project for Neuraville, gaining invaluable experience in the industry, and we are eager to see how the parser continues to develop and what it allows people to accomplish!
