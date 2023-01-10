
# OnlineFPS Sample - Building & Playing

In order to play the game, you need to either host a game, or connect to a game. This is done through the connection menu at the top-left of the screen when launching the game.

![](../_Images/onlinefps-connection-menu.png)


## Hosting a game

To host a game, you need to specify a port on which to listen to connections (the "Host Port"), and then click the "Host Game" button. This will start a server for the game, as well as a local client for you to play as.

NOTE: When hosting a game, if you want others to be able to connect, it is important that you **forward the port** you specified as your "Host Port" in your router settings. The exact procedure varies depending on the router. If you don't know how to do it, search for "port forwarding" + the brand of router you have.


## Connecting to a game

To connect to a game/server, you must enter the IP and port of the desired server in the "Join IP" and "Join Port" fields. Once this is done, press the "Join Game" button.

The "Join IP" is the public IP of the server. You can get the public IP by going to [https://www.whatismyip.com/](https://www.whatismyip.com/) on the computer that is hosting the game, and getting the "Public IPv4" address. However, if you try playing by launching multiple builds on the same machine, the "Join IP" should always be "127.0.0.1"

The "Join Port" is the port that the server specified as their "Host Port".


## Building the project

When trying to play the game online, especially with multiple game instances on the same machine, it might be preferable to build the game rather than playing in the editor. To do so, you can simply build the game normally using the `File > Build Settings` menu.