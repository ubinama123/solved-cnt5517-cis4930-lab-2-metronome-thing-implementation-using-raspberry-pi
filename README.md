Download Link: https://assignmentchef.com/product/solved-cnt5517-cis4930-lab-2-metronome-thing-implementation-using-raspberry-pi
<br>
In this assignment you will set up and (learn how to) program your Raspberry PI

3 Model B, while simultaneously preparing the device to become a Thing in the Internet of Things. Specifically, you will create a simple metronome using push buttons and LEDs on your Pi board, and then create a web interface to interact with the device remotely.




<h1>Background</h1>

A metronome is a device that creates a visual or audible pulse at consistent intervals. A common measurement for the speed, or tempo, of a metronome is Beats per Minute (BPM). A metronome at 60 BPM will pulse once every second, and at 120 BPM, twice every second, etc.




A common way to set the speed of modern metronomes is for the user to repeatedly press a button at the desired tempo. The metronome will calculate the average time between the presses and set the BPM accordingly. You will emulate this feature using one of two push buttons needed for this assignment.




<h1>Requirements</h1>

The following will be required for this assignment:

<ul>

 <li>Raspberry Pi 3 Model B and online manuals and programming tips.</li>

 <li>Your laptop</li>

 <li>One breadboard</li>

 <li>Jumper wires (male to male to use on the breadboard, and female to male jumper wires to connect your IO port on your Pi to the breadboard)</li>

 <li>Resistors</li>

 <li>Tne LED: one Green and one Red.</li>

 <li>Two pushbuttons</li>

 <li>USB-x to Ethernet adaptor cable to connect your Pi to your laptop. This way you will be able to use ssh to connect to your RPi from your laptop, which avoids having to connect a monitor, keyboard, mouse etc to the RPi. Also, UF WiFi may not allow you to do SSH from your laptop to your Pi, hence the tethered connection. To more information on this specific requirement see the additional document “<strong>Lab2-</strong><strong>Configuring your Raspberry Pi Remote Connection.pdf” </strong>placed in Canvases’ FILE section.</li>

</ul>




<h1>Part One — Embedded Application &amp; RESTful API</h1>

For the first part of this lab, you will create a metronome-style feature on your Pi board. The metronome should have two modes; a “<strong>play</strong>” mode, which will blink an LED at a given tempo, and a “<strong>learn</strong>” mode, in which the user will repeatedly press a button to set the tempo. The two modes will be toggled between using another dedicated push button.




In the “play” mode, the green LED will blink repeatedly at the previously specified tempo. On startup, the default mode should be at a tempo of 0, meaning the LED will always be off.




In the “learn” mode, the red LED will pulse every time the user presses the button, and the time of the tap will be recorded. Once the user is done tapping (i.e., the user presses the mode button again), the BPM should be calculated and the new value should be reflected in the “play” mode. At least 4 taps are required to create a valid BPM measurement.




To make your metronome a Thing, your board will expose the following REST endpoints with the respective CRUD operations:

<ul>

 <li>/bpm/ (GET, PUT): The current metronome BPM. Writing a new value here will update the speed the LED blinks at.</li>

 <li>/bpm/min/ (GET, DELETE): Return the lowest BPM the user has set the metronome to (not 0). On delete, reset this value to 0 (“none”).</li>

 <li>/bpm/max/ (GET, DELETE): Works the same as the min endpoint, but tracks the highest BPM the user has set.</li>

</ul>




<h1>Connecting the Hardware</h1>

The Raspberry PI interacts with the push buttons and LEDs through its GPIO (General Purpose I/O) pins. Each of these pins can read and write digital (on/off) values, controlled through the command line or a library. Each pin is numbered so that it can be accessed through the software; this numbering can be found <u>here</u>. Note that some pins are reserved for ground and power, and that there are multiple voltages available. ONLY USE THE 3.3V POWER PINS. The RPi GPIO senses at this voltage, so using 5V may damage the device.

To connect the LED, a circuit must be created between an output GPIO pin and a GND pin. Note that if you plug the LED in directly, it will attempt to draw as much current as possible, and probably burn itself out. Therefore, an appropriatelysized resistor must be connected in series with the LED. An exemplary circuit diagram can be seen below. Pay attention to the orientation of the anode/cathode legs of the LED.







To connect the push button, a circuit must be created with 3.3V on one side, and an input GPIO pin on the other. This way, once the button is pushed, the 3.3V (“HIGH”) signal is connected to the input pin. In this configuration, however, the pin has no way to sense when it is not pressed, since it has no connection to GND (“LOW”)—this is called a “floating” input, and will return noise (random HIGH/LOW) in this state. To make sure the button always reads LOW when it is not pressed, we need a pull-down resistor connecting it to ground. The resistor makes sure the GPIO pin is always connected, but will be “overridden” by the “un-resisted” HIGH signal when pressed. A circuit diagram is shown below.







Note that we also include a current-limiting resistor between the button and the GPIO pin. Before the RPi is configured, the GPIO may be set to output—pressing the button in this state might create a direct connection between power and ground without the protective resistor. <em>Also note that the RPi device contains built-in pull-up and pull-down resistors on each GPIO pin. These can be controlled through software and used in lieu of the physical 10k resistor. </em>




<h1>Getting Started</h1>

A skeleton project (<strong>metronome.zip</strong>) has been prepared to get you started with the RPi GPIO functionality. The program will continuously blink an LED as long as a push button is held down. Unzip the files included with the assignment onto your RPi. The sources are heavily commented with some insight on how to complete this lab. You will also need to install some additional packages:




sudo apt install wiringpi libcpprest-dev




Be sure to specify the LED and button pin numbers that correspond to your physical circuit setup. Once configured, you can compile the program with the following command (on your Raspberry Pi, inside the cloned folder):




mkdir build &amp;&amp; cd build  cmake ..      make




The program can then be run using this command:




./main

<strong> </strong>

Note that one subsequent builds, only make needs to be run. The sample also exposes a simple REST service that should be accessible through the same IP you use to SSH into your device, on port 8080.




<strong> </strong>

<h1>Implementing the Metronome</h1>

Note that the base project also contains the header metronome.hpp. This contains the definitions for the functionality of the metronome itself. You will need to write the implementation of this class. The metronome should start_timing() in the “learn” mode, and stop_timing() upon return to the “play” mode. While timing, each press of the button should record the current time. The m_beats array can be used to implement a running average—only the most recent <em>N</em> taps are recorded.




<h1>Part Two — HTML Dashboard Application</h1>

For this part of the lab, you will create a web dashboard application to interact with your embedded service. The application will allow you to easily interact with your REST endpoints.




Therefore, your web application should allow you to:

<ul>

 <li>Refresh the current BPM, along with the minimum and maximum.</li>

 <li>Set a new BPM on the device.</li>

 <li>Reset the current minimum and maximum BPM.</li>

</ul>




<h1>Getting Started</h1>

The dashboard should be implemented as a simple HTML and JavaScript project. It should present some simple styling to visualize the BPM values and allow the user to interact with the API through a set of buttons. No specific format or layout is necessary, but the user should be able to interact with all endpoints offered by the RPi through the dashboard.


