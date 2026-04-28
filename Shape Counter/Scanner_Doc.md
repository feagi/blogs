Blog Post

	Our project with Neuraville is a circuit capable of counting shapes from static input.  
The following is a description of the components of the circuit we used to accomplish this task.

Scanner:

	Let’s begin with a circuit which samples video input in the vision\_c cortical area by reading a small subset of neurons at a time and passing that subset on to later circuits for processing. Here, we will show you our implementation.

	First, we need a new cortical area where we will create this sampled video feed. I will refer to it as the vision sampler and it should be the same size as vision\_c. Our first connection will be from vision\_c to the vision sampler which should be made in the autogen region of the Brain Visualizer. This connection should be defined as block\_to\_block with default options. 

Next, we should think about how vision sampler should react to this new connection. Assuming you have firing neurons in the vision\_c our new cortical area should be an exact match if all is working properly. Instead, we want it to display only the neurons we are intending to sample. To make this happen we will take advantage of the Neuron Firing Parameters, in particular the Neuron Firing Threshold. Right now, vision\_c’s PSP should be set to 1.0. Let’s change vision sampler’s firing threshold to 36\. Now, you should see no neurons firing in our new region, this is because vision\_c’s signal simply isn’t strong enough to cross this threshold. The reason for this will come to light with out next cortical area.

The second cortical area we are creating will handle the logic of the parsing motion we are looking for. I will refer to it as the scanner cortical area. Place it in the autogen area and connect its output to the vision sampler with a projector connection. Next, set the scanner’s PSP to 35 and turn on PSP Uniformity. When the area’s 35 PSP is projected onto vision sampler it will add onto the neurons vision\_c is exciting with a PSP of 1, resulting in a total PSP of 36, crossing the vision sampler’s firing threshold, wherever the scanner is actively parsing.

Continuing with the scanner, the size of this area should change with your exact needs but I will be operating under the assumption of a 4x4x1 sized area where each neuron will be projected into a 32x32 block on the 128x128 vision sampler. Now, let’s edit the scanner’s cortical area details. Turn on MP Accumulation, set the firing threshold to 100, and set Consecutive Firing Count to 40\. We are now ready to implement the scanner's logic.

We will implement four recursive connections from the scanner to itself. 

1. Sweeper : This is a built-in connectivity rule in FEAGI that handles the core logic of scanning. Simply set it, leave the defaults as they are, and forget it.  
2. Last\_to\_first : Another built-in connectivity rule in FEAGI which ensures the sweeper movement repeats the full sequence when reaching the last neuron. Again, set with defaults and forget it.  
3. Block\_to\_block: Another built-in connectivity rule in FEAGI that we will not leave default. Once set, change its PSP multiplier to 10.0000.  
4. Inhibit\_prior: The final connection which will ensure only one neuron fires at a time, creating a clean scan of the entire area. This is a custom vector connectivity rule created with the following three vectors. Once implemented, set the PSP Multiplier to 10 and turn on the Inhibitory switch.  
   1. X=-1 Y=0 Z=0  
   2. X=3 Y=-1 Z=0  
   3. X=3 Y=3 Z=0

	That’s it for the recursive connections, now how should we start the scanning? We need to apply an initial bit of PSP to start the continuous process. Create a third cortical area which i will refer to as start, it should be a simple 1x1x1 area with PSP Uniformity turned on, a PSP of 255.0, a PSP Max of 35.00, and a Firing Threshold of 0.10. It should only have a single connection leading from it to scanner. It is a custom connection which we will call scan\_start. It is a Patterns-type connectivity rule with a source of \[\*, \*, \*\] and a destination of \[1, 1, 0\].

	Assuming all has gone well, to start the scanner all you have to do is manually select the one neuron in start with a shift+click and press the space bar to manually fire it. You should now see a scanning pattern of neuron firing in scanner. If it looks like multiple are firing at once, trying playing around with the Brain Visualizer's Refresh Rate. Start with 10Hz and go from there. 

	Well, done you should now have a scanner working. Now let’s actually extract the data we are scanning so other circuits can use it. Let’s make another cortical area called feature extraction. Make it 32x32x1 and leave all other options default. We will connect from vision sampler to feature extraction using the tile connectivity rule so that whatever is firing in vision sampler will fill the entirety of feature extraction. Now whatever potential is sent out from our scanning circuit will be easily handled by your other circuits.

Edge Extractor:

	The next piece of the shape counting circuit is edge extraction. For our use case we used shapes that were bold, causing all the neurons within the shape to fire as well as the outer edge. We found this distracting and would harm edge and curve detection so we set up this stage to ignore the inner fill and only pass on the edges to angle and curve detection.  
	  
	There are two cortical areas; an input area and an output area. The input area receives a projector connection from Scanner’s feature extraction area. There are then two exiting connections from input to output. The first is a simple block\_to\_block with a PSP multiplier of 12\. The second is a custom vector connection called lateral inhibition. It should have the following Vectors:

1. X: \-1 Y: \-1 Z: 0   
2. X: 0 Y: \-1 Z: 0   
3. X: 1 Y: \-1 Z: 0   
4. X: \-1 Y: 0 Z: 0   
5. X: 1 Y: 0 Z: 0   
6. X: \-1 Y: 1 Z: 0   
7. X: 0 Y: 1 Z: 0   
8. X: 1 Y: 1 Z: 0 

Now set the Inhibitory toggle to on and the connection is good to go. This vector connection is the core function of the edge extractor that only passes on power from neurons that aren’t surrounded by other firing neurons.

These next two circuits are somewhat complicated and have some redundancies, so we will focus on giving you the main idea of how it works while giving some technical details. See the completed shape counter to get full implementation details. 

Angle Detector:

	As we get into shape detection, we realise there are two main features that distinguish shapes from one another: angles and curves. 

	This circuit takes on the task of discerning if the input contains an angle that can then be used to discern the presence of a shape. First, we must accept input from the edge extraction circuit, in our implementation this area is called v1 and it simply takes a projection connection. V1 then passes onto v2 via a rotator\_z connection which is then passed onto many different areas filtered based upon the orientation of the angle we are attempting to detect. See, we want to know the angle of the intersection of lines, the exact orientation should not be relevant to whether or not it is a 90 degree angle for the purposes of the detector.  
	  
	Once rotated and fed into these separate areas depending on orientation, we begin the work of filtering the input based on whether or not a horizontal line or vertical line or diagonal line is present. Whatever passes through this filter is then sent to areas that will fire based on the combination of these lines. Have a horizontal and a vertical? That is a right angle and is passed on to the output area of the circuit. The right angles are the most important for our purposes but the circuit is also set up to pass on other angles of various combinations including two diagonals, a horizontal and diagonal, or a vertical and diagonal. The result is an output is sent once we have confirmed there is some form of connection of lines that form an angle in the input feed.  
	  
Curve Detector:

	Curve detection starts out similarly to angle detection; we accept an input from the edge extractor to v1 which is then passed onto v2 via a rotator\_z connection. This is then sent on to a large number of areas to filter for orientation of the curve we are analyzing. Each orientation filter then passes onto curve filters that pass onto the output depending on the angles we are looking for. We are interested in detecting squares so we handle the possibility of right angles in the input feed and pass them on. We also account for curves in the input since we are actively looking for circles as well. For these we created custom vector connectivity rules using the visual editor to detect curves of varying gradients. See the implementation for more exact details, “Steep\_Left\_Curve” for example; the vectors can be directly copied by hand for exact recreation.  We see an output once we have confirmed the presence of a right angle or a gradient, allowing for both squares and circles to be detected. 

Shape Detector:

	The shape detector area is very simple, it essentially acts as the area that actually confirms that a shape has been found. If input from the angle detection or curve detection or both reach here, then we can be reasonably sure that a shape has been found and can be passed on to the counting circuit.

	There are two input areas that accept projector inputs from the angle detector and the curve detector. Each one of these areas then projects into separate 1x1x1 areas called circle detector and square detector respectively that will be used as output connections for the counter.

Counter:

	The counter is made up of four cortical areas. Three of which are inside the main region while one will is a function of the Perception Inspector which we will use to extract an actual number for the count. 

	The input is a 1x1x1 area called shape detect which receives a projected input from both the 1x1x1 outputs of the shape detector. This area acts as a trigger that fires when a shape is detected. It uses two connections to the next region: a simple projector and a first\_to\_last set to a PSP multiplier of 10\.   
	  
	Next is the 1x1x10 region called the shape register which fills up with potential as the trigger repeatedly fires. A neuron on the register should fire each time the trigger fires. To accomplish this we set the area’s firing threshold to 2 and turn on PSP Uniformity. We then use two recursive connections to actually manifest the filling of the register. The first is a simple block\_to\_block connection, set to a PSP Multiplier of 2, that ensures the areas keep firing even when the trigger isn’t. The next is a lateral\_-z connection which ensures potential moves up the stack of neurons instead of building up in just one.   
	  
	The next area is another 1x1x10 meant to only fire the neuron with the z-value equal to the count that will be turned into a number. So if one shape has been detected, only z=0 will fire. Then if another is found then z=0 will deactivate and z=1 will fire instead and so on as the count accumulates to a maximum of z=9 or a count of 10\. We called it the shape count and it works with two output connections from the shape register; a block\_to\_block and a lateral\_+z set to a PSP Multiplier of 10 and Inhibitory toggled on. This creates the moving neuron we are looking for.

	Next is the area called Count Output 0-0 which is created and used by the Perception Inspector feature in Neaurobotics Studio. Simply use the block\_to\_block connection from shape count to count output so that the inspector can see the count. Then, simply open the Perception Inspector and scroll down to the Count Thoughts area where we will see an incrementing number as objects are found.

