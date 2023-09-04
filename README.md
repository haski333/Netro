
# Netro

**Netro** is a roblox framework/library that makes internal communication between the **server** and the **client** easier and safer.

You can install **Netro** [here]()

## Getting started

Let's get started with the base features of **Netro**  
**Netro** has a type of module you can create that is called a **"shared module"**

A **shared module** is just a module that contains the following components:
* A server table
* A shared table
* A client table

Every function inside the **shared table** will be shared to the **client** and the **server**  
Everything inside the **client table**/**server table** will be private to each side accordingly

Here is an example of a coin system we can make with **Netro**

* Module script (ReplicatedStorage)
```lua
local Netro = require(game.ReplicatedStorage.Netro)

-- Create the shared module
local CoinsSystem = Netro.CreateSharedModule {
    ModuleName = "CoinsSystem"
}

-- Create the table where we will be holding our players coins
local PlayersCoins = {}

-- Here we create a shared method which will automatically receive "Player"
function CoinsSystem.Shared.GetCoins(Player)
    local Coins = PlayersCoins[Player] -- Get coins
    return Coins or 0 -- Return coins or 0 if no coins were found
end

-- Here we create a server method that will add coins to a player
function CoinsSystem.Server.AddCoins(Player, Ammount)
    local Coins = CoinsSystem.Shared.GetCoins(Player) -- Get coins
    Coins += Ammount -- Add the coins

    PlayersCoins[Player] = Coins -- Set the player's coins
end

-- Export the shared module
return CoinsSystem:Export()
```

* Script (ServerScriptService)
```lua
local CoinsSystem = require(game.ReplicatedStorage.CoinsSystem)

-- Create a player added function
local function PlayerAdded(Player)
    CoinsSystem.AddPoints(Player, 10) -- Add 10 points to the player
end

local Players = game:GetService("Players")
Players.PlayerAdded(PlayerAdded) -- Connect the function the the event
```

* LocalScript (StarterPlayerScripts)
```lua
local CoinsSystem = require(game.ReplicatedStorage.CoinsSystem)
local Coins = CoinsSystem.GetPoints() -- Get points (When we call GetPoints our client's player will automatically be received on the server)

print(CoinsSystem.GetPoints()) -- 10
```

Now, let me clear some things up.  

To use a **shared** function on the client, the **server** needs to require the module for the function to be initialized.  

Under the hood, **Netro** takes care of the remote objects that are used to actually create the **server**/**client** bridge so for **Netro** to actually create those remote objects, the server needs to require the module so **Netro** can process the functions

In every roblox game, the **server** loads faster than the **client** but for some reason,
if in your script, you yield some time and then require the module so that the **client** loaded faster and called **shared** methods that weren't initialized, don't worry, **Netro** will yield the called function until the **server**/**client** bridge is created and call the function

## Safety

Let's also start talking a bit about the safety of **shared modules**  
As you saw in our coins system code snippet, we have our coins system module inside **ReplicatedStorage**

"So, what's the problem?"  
Well, that gives people the possiblity of stealing **server** code as everything inside **ReplicatedStorage** is, replicated😁! Including scripts...😔 and we don't really want that.

This shouldn't really be a problem as not all of your **server** modules will be in ReplicatedStorage

But if you really wanna keep your **server** code private and protected from being copied, you have a way to do that

In the module settings you pass when making your **shared module**/**system** you have a setting called **ExternalServerModule** and ill show you how to use it

To start, we are gonna place a module script inside **ServerStorage** called **CoinsSystemServer**

```lua
-- Make a table with our server tables "Shared" and "Server"
local CoinsSystemServer = {
    Shared = {},
    Server = {}
}

-- This is just a copy from the original coins system snippet
local PlayersCoins = {}

function CoinsSystemServer.Shared.GetPoints(Player)
    return PlayersCoins[Player] or 0
end

function CoinsSystemServer.Server.AddPoints(Player, Points)
    PlayersCoins[Player] = CoinsSystem.Shared.GetPoints(Player) + Points
end

return MySystemServer
```

Now that we have this module and it's inside **ServerStorage**

Let's make a new module inside **ReplicatedStorage** just like we did in the original snippet

```lua
local Netro = require(game.ReplicatedStorage.Netro)

-- Get the ServerStorage and create a variable for our CoinsSystemServer module
local ServerStorage = game:GetService("ServerStorage")
local CoinsSystemServer = nil

-- We check if it's the server that is in this module, we do this so the client doesn't try and index something in ServerStorage because it can't
if Netro.IsServer() then
    CoinsSystemServer = ServerStorage.CoinsSystemServer -- Set the variable to the module
end

-- Create the shared module
local CoinsSystem = Netro.CreateSharedModule {
    ModuleName = "CoinsSystem"
    ExternalServerModule = CoinsSystemServer -- Set the external server module to our server module
}

-- Export the shared module
return CoinsSystem:Export()
```

Now because the module is contained in **ServerStorage** the client has no access to it and now our server code is safe.
We can still call all shared methods of the system on the client















