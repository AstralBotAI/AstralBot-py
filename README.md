# AstralBot-python


python3 implementation of the astralbot connector and a tool to load a python script

This library can be used to interact with an AstralBot instance.  
It will run the logic to play the game selected on the AstralBot.

The astral bot will propagate events throught websocket, each time a message is received the same function is called.  
You must implement this function and provide it to the AstralBot connector.  
This function will take an 'engine' parameter which represent the current state of the game engine and an 'info' parameter.  
This last one represent the information on the current running game like the screen data and game state.  
In response of each call to the function you can return an object representing the action to take.  
In the future you will be able to return a list of actions.

The game engine expose some API throught the websocket message protocol representing available actions.
Here is the list:

|   API   | args |     description                                           |
|---------|------|-----------------------------------------------------------|
| ping    |      | ping the engine to trigger again .state and .info         |
| start   | name | start the game with the name 'name'                       |
| clickAt | x, y | make the mouse click somewhere in the screen in info.data |

An example is available in astralbot-example.py

AstralBot lifecycle
-------------------

Application start (uninitialized) -> game engine init (initialized) => you are now allowed to start a game (the list is in engine.games).  
Game Engine start a game (starting) -> game process is started (started) -> game is configuring (ready) => your are allowed to play.

During the transition between started and ready a script can configure the game to change the difficulty or to join a server.  
The screen is sent according to the game script parameters (500ms in minesweeper_beginner). This information can be found in the 'game' object.  
The goal is to implement all the logic to play the game remotly.

How to implement your bot?
=========================

The first thing to do is to import the library and to set the AstralBot instance address

```
AstralBot = from astralbot import AstralBot
astralUrl = 'localhost:8080'
```

And to instanciate the astralbot connector you will call this:

```
options = { "verbose": True }
ab = AstralBot(astralUrl, options, loop)
// You can save all received screens in the disk.
// Notes: each screenshot will be replaced at each run.
// ab.enable_screen_saving()
// or
// ab.enable_screen_saving("./directory_for_screenshots")
//
// In the case the astral engine is protected you can add authentication informations
// ab.set_auth("myBot", "v1.0.0", "t0ken")
//
// Then run the bot:
ab.run()
```

Note that you can enable the screen saving mecanism to keep each screenshots sent by the astral engine.  
They will be save in a screenshot directory at the same level of your script.  
Also note that the bot will try to connect every second at startup to let the engine starts.

As you can see the AstralBot constructor take a `loop` parameter (name it like you want).  
It should be a function which will be called at each event sent by the astral engine.  
This function takes 2 parameters:

* `engine`: the current engine state (see lifecycle) and informations (list of supported games)
* `game`: the current game state and information (player status, screen data, list of available commands)

This function should return an object representing the action to ask to the astral engine.

An exemple of implementation could be:

```
def loop(engine, game):
  action = {}
  print(" * EngineState:", state)
  print(" * GameInfo:", game)

  return action
```

In this function you will start the targetted game when the astral engine become ready and you will decide which action to ask to the engine.

You can find a full example [here](src/astralbot-example.py).

Where to find an astral instance?
=================================

The astral engine is available as a docker image or (soon) a desktop application.
You will find all information [here](https://github.com/AstralBotAI/AstralBot-engine).

Where to find the community?
============================

A discord is availble [here](https://discord.gg/Xq33rrHFue).
