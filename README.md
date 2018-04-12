# Wago Documentation
for use with the [Utopia Smart City Console](https://github.com/byutopia/Capstone)

## Wago Device

For this project, we received the Wago Ethernet Starterkit, which includes the following components:

 - The ethernet controller: 750-880
 - A 2-channel digital input module: 750-400
 - A 2-channel digital output module: 750-501
 - An end module, required to terminate the sequence: 750-600
 - A 24V DC power supply: 787-1602
 - A CD with the CoDeSys software, required to program the controller
 
More information can be found on the official Wago web page for this kit, [located here](https://www.wago.com/global/plcs-controllers/starterkit/p/8003-001_K999-9999_000-1700). Included in the appendix of this documentation is the quickstart guide and the full manual for using the 750-880 ethernet controller.
To configure the network connection for the ethernet controller, please refer to section 3.2.3 of the quickstart guide.

## CoDeSys Software

The CD included in the Starterkit contains the CoDeSys software, which was difficult to learn on our own. To install the CoDeSys software:

 - Insert the CD (or mount the ISO image) and run `Setup.exe` to begin
 - Proceed through the dialogs
 - On the Select Components dialog, no additional components are necessary; continue without changing anything
 - Proceed until the software finishes installing
 
The CoDeSys software should now be available on your system. You will now be able to open and view the program created for this project, included in the appendix as `UTOPIA.pro`. Upon opening, there may be an error that a library is missing. Close the dialog(s) to finish opening the project.
To fix the missing library:

 - In the appendix of this documentation, locate the file named `WagoLibHttp_02.lib`. This library, obtained from the official Wago website ([here](http://www.wago.us/support/download-wizard/overview/index-2.jsp?q=WagoLibHttp_02#appnotedetails9166769170854579093)), enables the ethernet controller to send HTTP requests.
 - Copy this library to the same directory as the `UTOPIA.pro` file.
 - In the CoDeSys software, at the bottom of the left-side navigation pane, select the Resources tab (the right-most tab)
 - Double-click on Library Manager to open the library configuration panel
 - In the center-top pane, which lists the loaded libraries, there should be a line in red. This is the missing library. Right-click this line and choose Delete, and confirm the option.
 - In the empty space below the last library, right-click and choose Additional Library
 - Navigate to and select the `WagoLibHttp_02.lib`
 - The library is now loaded into the project, and should be available to the program. To return to the program interface, select the POUs tab from the bottom of the navigation pane and double-click one of the programs.
 
To reconfigure the connection to the ethernet controller:

 - Open the Online menu from the top of the window
 - Select Communication Parameters
 - Change the IP Address in the Address field to match the address of the ethernet controller
 
At this point, if the ethernet controller has been configured to have an address on the same network as the system, the CoDeSys software should be able to run programs on the device. To test the connectivity of the device, select Online > Login. If it successfully connects, the interface of the program should change to reflect the current state of the variables on the system. To run the current program, select Online > Run. To load the current program as the startup program for the controller, so that the program will run automatically when the device is powered on, select Online > Create boot project.

The program view in the CoDeSys software is different from many other programming environments: the two programs in our project are the the Function Block Diagram language. Visually, the variables are defined in the text editor on top and the logic is designed with blocks on the bottom. For more information, refer to the CoDeSys manual included in the appendix.

## Program Description

Within the `UTOPIA.pro` project, there are two programs: PLC_PRG, which is the main program that runs automatically; and API, the program that provides the monitoring functionality to the Smart City Console. As an analogy, think of PLC_PRG as filling the same role as the main() function in C or C++. In this project, all PLC_PRG does is call the API program. API has the main logic, and took the longest amount of time to complete.
The first three blocks in the logic chain determine whether or not to send a request to the web server. The fourth block contains the block that sends the request. The final block provides power to the output. The diagram below illustrates the purpose of these blocks.

![logic diagram](https://raw.githubusercontent.com/byutopia/wago-docs/master/API%20logic.png)

The pseudocode equivalent would be:
```
bOne = (xDi2 AND NOT(xDo1))
bTwo = (NOT(xDi2) AND xDo1)
xPostStart = (bOne OR bTwo)
```

Note that, in the project and in the diagram above:

 - **xDi2** is the state of the input, i.e. the position of the switch (on = true, off = false)
 - **xDo1** is the state of the output, i.e. is power being sent to the lights
 - **bOne** and **bTwo** are the placeholder variables for the results of the logic gates
 - **xPostStart** is the boolean variable that determines whether or not to send the request
 
To help the reader understand the logic, consider the following scenario:

1. Initially, the switch is in the off position, and no power is being sent to the lights. **xDi2** and **xDo1** are both false, neither AND gate is triggered, **bOne** and **bTwo** remain false, and thus **xPostStart** is also false. 
2. A user moves the switch into the on position. On the next cycle, **xDi2** becomes true, and since **xDo1** is still false the first AND gate is triggered, **bOne** becomes true and causes **xPostStart** to become true. An HTTP request is sent to notify the web server of the new state. Finally, the state of the switch, **xDi2**, is assigned to the output, **xDo1**, which supplies power to the lights, causing **xDo1** to become true. 
3. When the next cycle begins, **xDi2** is true, as the switch is still in the on position, and **xDo1** is true, neither of the AND gates trigger, and no request is sent. 
4. If a user then moves the switch to the off position, **xDi2** becomes false, and since **xDo1** is still true, the second AND gate is triggered, and **bTwo** becomes true, causing **xPostStart** to be true and sending the HTTP request. The value of **xDi2** is then assigned to **xDo1**, removing the supply of power and turning off the lights.
