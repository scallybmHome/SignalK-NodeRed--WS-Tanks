# SignalK-NodeRed--WS-Tanks
SignalK with remote node-red WS tank display

![image](https://user-images.githubusercontent.com/38928974/111848219-caf73480-88c7-11eb-9cf3-d1736aa6d10d.png)

**Aims of this project**

1 - display the present tank levels on the vessel.

2 - sit on any Node-Red instance capable of accessing the SignalK server.

3 - Make the connection over the Web Socket interface using the delta format.

4 - actually document the work.

**Explanation of the operation**

First thing to note this is a folded flow, I did this to make it easier to read.
There are a lot of debug points, these were usefull in development so I left them in.  Several of them have filters in-fromt of them to reduce interface bogging.

_Setup_

![image](https://user-images.githubusercontent.com/38928974/112069880-dfd30280-8b29-11eb-8bbd-a99e21659032.png)

The node red plugin knows where the server is, so the only configuration is the desired keys.  In this case 'tanks.*'.

![image](https://user-images.githubusercontent.com/38928974/111847755-b7979980-88c6-11eb-8d76-069e47fe0282.png)
The nodes underlined in purple configure the human readable name of the tank and tie it to the tank sender serial number.
In the real implementation this information is read from a JSONfile, as that is easier to edit dynamically.

![image](https://user-images.githubusercontent.com/38928974/111847871-034a4300-88c7-11eb-8b3b-db541905fb6a.png)
The node underlined in blue is a filter to ensure the SignalK deltas from my brefered N2K->SignalK gateway are processed.
It is likely that you get the information you want to display into your server from a different interface this block will need to be changed for your installtion.

_Flow_
![image](https://user-images.githubusercontent.com/38928974/112070294-9d5df580-8b2a-11eb-8d43-007abd13d075.png)

1 - This node opens the websocket interface to the server.  Setting the subscribed streams to the desired path.


6 - This run once flow branch sets up the tank specific infomration.  In the real system this reads from a JSONfile, but having this infomration stored on the SignalK server would be better.

7 - The recieved the delta messages are parsed to JSONobjects and past on to (8) through the link nodes.

8 - This node filters out all the entries not coming from my Prefered NMEA interface, this is purely to reduce workload, but it is good practise.  You will likley have modified this node to get the system to work.

9 - Tanks have 2 properties currentLevel and capacity.  These are sorted so only currentLevel causes a guage update.  capacity could be viewed as not a true delta and so is routed to be stored in the tank specific infomration.  On N2k tank sensors capacity is configured at the tank level sender,  so it makes sence is it part of the flow.

10 - The capacity values are used to update the tank specific infomration.  This is a grand set of if statment as it was easier to read than sort and assign blocks.

11 - This node looks at the update time of the last piece of information it has recieved and makes an dashboard LED red or green.  This would be better on a timer - next rev.

12 - These nodes combine the tank currentlevel values with the tank specific infomraiton to display guages that change colour when you are about to run out of something.
The nodes aport processing if the information is not for their paired guage so a delta expressed on the fuel tank does not cause all the gauges to be redrawn.
