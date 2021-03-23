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

**__NOTE__ The operation on a remote server complicates the operation.  IF you want to start this flow on a plugin node red installation see https://github.com/scallybmHome/SignalK-NodeRed--WS-Tanks/tree/embedded**

_Setup_
![image](https://user-images.githubusercontent.com/38928974/112067874-0abb5780-8b26-11eb-808a-ae98be3af7b8.png)
The node red WS interface nodes (underlined in red) are configured with the IP address and port of my SignalK server.
You will almost certainly need to modify this.  Common options include localhost:3000 for running locally.
__You should ONLY edit the left most interface.__
Double click on the node, click the pen icon and edit you SignalK server into the URL field.
It will be of the form - ws://signalKserver:3000/signalk/v1/stream?subscribe=none
Configuring the right block is as simple as double clicking on the box and using a pull down menu to  selecting the same name as you entered in the left block.
AGAIN do NOT be tempted to type the information in.
That seems to be the way to make it not work -- so I left it alone.

The nodes underlined in purple configure the human readable name of the tank and tie it to the tank sender serial number.
In the real implementation this information is read from a JSONfile, as that is easier to edit dynamically.

The node underlined in blue is a filter to ensure the SignalK deltas from my prefered N2K->SignalK gateway are processed.
It is likely that you get the information you want to display into your server from a different interface this block will need to be changed for your installtion.

_Flow_
![image](https://user-images.githubusercontent.com/38928974/112068226-b795d480-8b26-11eb-8ef2-abbc07fed8ec.png)
1 - This node opens the websocket interface to the server.  Setting the subscribed streams to none. If this is set to 'self' you may have problems.

2 - To function properly the msg_session message content returned from (1) must be moved or deleted.  This infomration confuses the following WS block. This is odd and rather couner intuative.  But it only works when the infomraiton is removed.

3 - This simple branch to detects if this is a "Hello" message marking this as the beginning of the communication.  It this is a 'hello' the flow passes to the setup sections.  See - http://signalk.org/specification/1.5.0/doc/streaming_api.html#connection-hello

4 - Generate a subscribe JSON. See - http://signalk.org/specification/1.5.0/doc/subscription_protocol.html If you want to select multiple data elements you can call them out individually or use a wildcard.  Having just 'tanks' does not work.

5 - This node passes the subscribe message back to the SignalK server starting the flow of information.

6 - This flow branch sets up the tank specific infomration.  In the real system this reads from a JSONfile, but having this infomration stored on the SignalK server would be better.

7 - The recieved the delta messages are parsed to JSONobjects and past on to (8) through the link nodes.

8 - The switch reformats the json delta letters into an 'identical' form to what would be derived from the signalk-node-red plugin.

9 - This node filters out all the entries not coming from my Prefered NMEA interface, this is purely to reduce workload, but it is good practise.  You will likley have modified this node to get the system to work.

10 - Tanks have 2 properties currentLevel and capacity.  These are sorted so only currentLevel causes a guage update.  capacity could be viewed as not a true delta and so is routed to be stored in the tank specific infomration.  On N2k tank sensors capacity is configured at the tank level sender,  so it makes sence is it part of the flow.

11 - The capacity values are used to update the tank specific infomration.  This is a grand set of if statment as it was easier to read than sort and assign blocks.

12 - These nodes combine the tank currentlevel values with the tank specific infomraiton to display guages that change colour when you are about to run out of something.
The nodes aport processing if the information is not for their paired guage so a delta expressed on the fuel tank does not cause all the gauges to be redrawn.

13 - The node in the primary flow adds the time stamp of the latest readings to flow.local information structure.  The timer inintiated part of the flow compares present time to the last reading values.  This allows the LED to go red when no messages arrive for a long time.

14 - This flow toggled between relative and absolute tank fills.  The guages don't update till a message arrives.
