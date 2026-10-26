# Embedded Linux Reaction Timer

The source code is private at my instructor’s request. This repository contains the project overview, wiring diagram, hardware photos, and demo video.

For my first embedded Linux project, I put together a joystick reaction game on a BeagleY-AI running Debian Linux.

The player responds to a random LED cue: green means push UP, and red means push DOWN. The program measures the response time and keeps the fastest correct result for the session.

Getting it running involved connecting the hardware, setting up a Debian development environment, and building C code to run on the board.

![Reaction timer wiring diagram](images/reaction_timer_wiring.svg)
![BeagleY-AI reaction timer hardware](images/hardware.jpg)
[Watch the reaction timer demo](LinuxReactionTimerDemo_GitHub.mp4)
### Hardware and Tools

- BeagleY-AI: Runs the application and controls the LEDs
- Analog joystick: Provides X and Y voltages
- MCP3208 12 bit ADC: Converts the voltages into digital readings
- Breadboard and jumper wires: Connect the hardware

The application uses C, CMake, Linux spidev, and sysfs LED controls. I worked in a Debian VM running in UTM on a Mac, using SSH to access the board and NFS to share the executable.

### How the Game Works

Each round starts with alternating green and red flashes. If the player is holding UP or DOWN, the program waits for them to release it.

After a random delay of 500 to 3000 milliseconds, the program checks for an early UP or DOWN input. If one is detected, it prints “Too soon!” and restarts the round.

Otherwise, it chooses a direction, turns on the matching LED, and starts timing.

A correct response produces five green flashes and prints the response time. An incorrect response produces five red flashes.

Moving LEFT or RIGHT at the response cue exits the game. Waiting more than five seconds without responding also ends the session.

### Reading the Joystick

The joystick's X and Y outputs connect to channels 0 and 1 of the MCP3208. The program reads these channels over SPI and receives values between 0 and 4095.

The joystick module uses thresholds of 800 and 3300 to determine direction. Readings from 800 through 3300 count as neutral for that axis. X is checked first, so horizontal movement takes priority during diagonal input.

This keeps the ADC and SPI details inside the joystick module. The game receives a direction and decides what to do with it.

### Software Structure

The program is split into separate parts for the game, joystick, and LEDs. The game handles the rounds and response timing. The joystick module reads movement, while the LED module controls the visual feedback.

### Development Setup

I followed setup guides to configure the Debian VM, USB networking, SSH, and NFS.

The VM builds the program using a compiler that builds programs for the Beagle board. CMake links the application with the separate joystick and LED modules, then copies the executable into the host's shared folder.

The board accesses that folder through its NFS mount at `/mnt/remote`. I used SSH to run the executable on the board.

The host and target paths refer to the same shared file. After changing the source, I rebuild the program before running the updated version.

### Problems I Worked Through

**Learning to use Linux**

Using Linux for the first time was incredibly confusing and took a lot of time. Even basic tasks, like moving between folders, running commands, and figuring out where files were saved, required explanations and practice.

I also had to keep track of whether I was working on the Debian VM or the Beagle board. I followed guides and worked through errors with help, gradually making more sense of what each step was doing.

**Getting SPI working**

At first, Linux did not show the SPI connection that the program needed to read the ADC. I struggled to understand what was missing and how to enable it.

Following the setup instructions, I changed the board's startup settings and restarted it. After that, the SPI connection appeared as `/dev/spidev0.0`, allowing the program to communicate with the ADC.

**Getting the shared folder working**

Setting up the shared folder between the VM and board was another confusing part of the project. The first attempts failed, and I needed help working through the setup and error messages.

The board was missing software needed to access the folder. Installing it and working through the network setup eventually let the board access the program built on the VM.

**Program not found at login**

During rehearsal, the board tried to launch the game but reported that it could not find the program. This was confusing because I had already built it.

I traced the problem to the shared folder being unavailable on the board. Running the script that reconnects the folder restored access to the program.

These problems took repeated attempts and explanations to work through. Getting the game running involved learning how to use the development setup alongside learning the code.
### Testing

I tested the application on the physical board during development and rehearsal.

- Correct direction: Green feedback and response time
- Incorrect direction: Red feedback
- UP or DOWN held through preparation: Wait for release
- UP or DOWN held at the early input check: Restart the round
- No response: Exit after the timeout
- LEFT or RIGHT at the response cue: Exit normally
- Faster correct response: Update the best time
- Slower correct response: Keep the previous best time

The recorded terminal session shows a correct response, an incorrect response, and exit through joystick input. The remaining cases were checked manually during rehearsal.

### What I Learned

This project gave me a practical way to understand how C code interacts with hardware through Linux.

I worked through how joystick voltages become ADC readings, how SPI carries those readings, and how the program converts them into directions. Reviewing the code also helped me understand pointers, enums, file descriptors, and the state shared within each module.

Keeping track of the VM and board took practice. Checking the terminal hostname helped me identify where commands were running and which files they could access.

### Current Limits

Response detection uses polling and includes delays from Linux and the time needed to read the joystick. The five millisecond polling pause does not guarantee five millisecond timing accuracy.

The direction thresholds are fixed. Early input is checked once after the random delay, so a brief movement released before that check can be missed.

The best result resets when the application closes.

The build copies the executable to my NFS shared folder. Other users would need to change this destination to match their setup.

### Acknowledgments

Rodrigo Villalon

Developed for SFU ENSC 351 using the supplied starter and technical guides. AI tools assisted troubleshooting and explanations.
