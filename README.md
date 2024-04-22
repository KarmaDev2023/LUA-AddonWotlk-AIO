# AIO
AIO est un pur système de communication serveur-client Lua pour Eluna et WoW.
AIO est conçu pour envoyer des modules complémentaires Lua et des données au lecteur depuis le serveur et des données du lecteur au serveur. 
Fabriquer pour [Eluna Lua Engine](https://github.com/ElunaLuaEngine/Eluna). Testé sur 3.3.5a et devrait fonctionner sur d'autres correctifs. Testé avec Lua 5.1 et 5.2. 
[Third party C++ support is made by SaiFi0102](https://github.com/SaiFi0102/TrinityCore/blob/CAIO-3.3.5/CAIO_README.md). Il vous permet d'utiliser AIO sans avoir besoin d'Eluna.

Backlink: https://github.com/Rochet2/AIO

# Installation
- Assurez-vous que vous avez [Eluna Lua Engine](https://github.com/ElunaLuaEngine/Eluna)
- Copiez le `AIO_Client` à ton `WoW_installation_folder/Interface/AddOns/`
- Copiez le `AIO_Server` à ton `server_root/lua_scripts/`
- Voir les paramètres de configuration sur le fichier AIO.lua. Vous pouvez modifier respectivement le fichier serveur et client
- Lors du développement d'un module complémentaire, il est recommandé de désactiver AIO_ENABLE_PCALL et parfois vous aurez peut-être besoin d'activer AIO_ENABLE_DEBUG_MSGS pour voir des informations sur ce qui se passe..

# À propos
AIO fonctionne de manière à ce que le serveur et le client disposent de leurs propres scripts Lua qui gèrent l'envoi et la réception de messages entre eux.
Lorsqu'un module complémentaire est ajouté à AIO en tant que module complémentaire à envoyer au client, il sera traité (en fonction des paramètres, masqué et compressé) et stocké en mémoire en attendant d'être envoyé aux joueurs.
Tous les modules complémentaires ajoutés sont exécutés côté client dans l'ordre dans lequel ils ont été ajoutés à AIO.
AIO utilise un système de cache pour mettre en cache les codes complémentaires côté client afin qu'ils n'aient pas besoin d'être envoyés à chaque connexion.
Ce n'est que si un module complémentaire est modifié ou ajouté que le nouveau module complémentaire est renvoyé. L'utilisateur peut également vider son cache AIO local, auquel cas les modules complémentaires seront à nouveau envoyés.
Le code complémentaire complet envoyé au client est exécuté sur le client tel quel. Le code a un accès complet à l’API du module complémentaire côté client.
La messagerie client-serveur est gérée avec une classe d'assistance de message AIO. Il contient et gère les données à envoyer.

# Commandes
Certaines commandes peuvent être utiles.
Côté client, utilisez `/aio help` pour en voir une liste. Côté serveur, utilisez « .aio help » pour en voir une liste.

# Sécurité
La messagerie entre le serveur et le client est codée pour être sécurisée

- vous pouvez limiter la taille du cache, les délais et autres dans AIO.lua
- les données reçues du client sont uniquement désérialisées - pas de compressions, etc.
- la bibliothèque de sérialisation n'utilise pas de chaîne de chargement pour sécuriser la désérialisation
- lors de la réception de messages, le code est exécuté dans pcall pour éviter que toutes les données envoyées par l'utilisateur ne créent des erreurs. Activez les messages de débogage dans AIO.lua pour voir également toutes les erreurs côté serveur
- le code est aussi sûr que vous le créez. Dans vos propres codes, assurez-vous que toutes les données que le client envoie au serveur et que vous utilisez sont du type que vous attendez et se situent dans la plage dans laquelle vous vous attendez. (exemple : math.huge est un type numérique, mais pas un type numérique. nombre réel)
- assurez-vous que votre code a des assertions en place et qu'il est rapide. Il y a un délai d'attente modifiable dans AIO.lua juste pour être sûr que le serveur ne se bloquera pas si vous écrivez du code incorrect ou abusif ou si un mauvais utilisateur trouve un moyen de bloquer le système.
- Vérifiez les paramètres AIO.lua et adaptez-les à vos besoins respectivement pour le client et le serveur. Ceci est important pour repousser les mauvais utilisateurs et améliorer le fonctionnement de votre configuration.

# Gestionnaires
AIO a quelques gestionnaires par défaut qui sont utilisés pour les codes internes et vous pouvez
utilisez-les si vous le souhaitez.
Vous pouvez également coder vos propres gestionnaires et les ajouter à AIO avec les fonctions décrites dans la section API. Voir AIO.RegisterEvent(name, func) et AIO.AddHandlers(name, handlertable)

```lua
-- Forcer le rechargement de l'interface utilisateur du joueur
-- Affiche un message indiquant que l'interface utilisateur est rechargée de force et recharge l'interface utilisateur lorsque le joueur
-- clique n'importe où sur son écran.
AIO.Handle(joueur, "AIO", "ForceReload")

-- Forcer la réinitialisation de l'interface utilisateur du joueur
-- Réinitialise les variables enregistrées du module complémentaire AIO et affiche un message indiquant que l'interface utilisateur est forcée
-- rechargé et recharge l'interface utilisateur lorsque le joueur clique n'importe où sur son écran.
AIO.Handle(joueur, "AIO", "ForceReset")
```

# API
Pour des exemples de scripts, consultez le dossier Exemples. Les fichiers d'exemple sont nommés en fonction de leur emplacement d'exécution final. Pour exécuter les exemples, placez tous leurs fichiers dans `server_root/lua_scripts/`.

There are some client side commands. Use the slash command `/aio` ingame to see list of commands

```lua
-- AIO est requis de cette façon en raison des différences entre le serveur et le client avec la fonction requise
AIO local = AIO ou require("AIO")

-- Renvoie vrai si nous sommes côté serveur, faux si nous sommes côté client
isServer = AIO.IsServer()

-- Renvoie la version AIO - notez que le type n'est pas garanti comme étant un nombre
version = AIO.GetVersion()

-- Ajoute le fichier au chemin donné aux fichiers à envoyer aux joueurs s'il est appelé côté serveur.
-- Le code complémentaire est coupé en fonction des paramètres dans AIO.lua.
-- L'addon est mis en cache côté client et sera mis à jour uniquement en cas de besoin.
-- Renvoie false côté client et true côté serveur. Par défaut le
-- path est le chemin du fichier actuel et name est le nom du fichier
-- 'path' est relatif à worldserver.exe mais un chemin absolu peut également être donné.
-- Vous devez appeler cette fonction uniquement au démarrage pour vous assurer que tout le monde obtient la même chose
-- addons et aucun addon n'est en double.
ajouté = AIO.AddAddon([chemin, nom])
-- La façon dont ceci est conçu pour être utilisé se trouve en haut d'un fichier complémentaire afin que le
-- le fichier est ajouté et n'est pas exécuté si nous sommes sur le serveur, et simplement exécuté si nous sommes sur le client :
si AIO.AddAddon() alors
    retour
fin

-- Similar to AddAddon - Adds 'code' to the addons sent to players. The code is trimmed
-- according to settings in AIO.lua. The addon is cached on client side and will
-- be updated only when needed. 'name' is an unique name for the addon, usually
-- you can use the file name or addon name there. Do note that short names are
-- better since they are sent back and forth to indentify files.
-- The function only exists on server side.
-- You should call this function only on startup to ensure everyone gets the same
-- addons and no addon is duplicate.
AIO.AddAddonCode(name, code)

-- Triggers the handler function that has the name 'handlername' from the handlertable
-- added with AIO.AddHandlers(name, handlertable) for the 'name'.
-- Can also trigger a function registered with AIO.RegisterEvent(name, func)
-- All triggered handlers have parameters handler(player, ...) where varargs are
-- the varargs in AIO.Handle or msg.Add
-- This function is a shorthand for AIO.Msg():Add(name, handlername, ...):Send()
-- For efficiency favour creating messages once and sending them rather than creating
-- them over and over with AIO.Handle().
-- The server side version.
AIO.Handle(player, name, handlername[, ...])
-- The client side version.
AIO.Handle(name, handlername[, ...])

-- Adds a table of handler functions for the specified 'name'. When a message like:
-- AIO.Handle(name, "HandlerName", ...) is received, the handlertable["HandlerName"]
-- will be called with player and varargs as parameters.
-- Returns the passed 'handlertable'.
-- AIO.AddHandlers uses AIO.RegisterEvent internally, so same name can not be used on both.
handlertable = AIO.AddHandlers(name, handlertable)

-- Adds a new callback function that is called if a message with the given
-- name is recieved. All parameters the sender sends in the message will
-- be passed to func when called.
-- Example message: AIO.Msg():Add(name, ...):Send()
-- AIO.AddHandlers uses AIO.RegisterEvent internally, so same name can not be used on both.
AIO.RegisterEvent(name, func)

-- Adds a new function that is called when the initial message is sent to the player.
-- The function is called before sending and the initial message is passed to it
-- along with the player if available: func(msg[, player])
-- In the function you can modify the passed msg and/or return a new one to be
-- used as initial message. Only on server side.
-- This can be used to send for example initial values (like player stats) for the addons.
-- If dynamic loading is preferred, you can use the messaging API to request the values
-- on demand also.
AIO.AddOnInit(func)

-- Key is a key for a variable in the global table _G.
-- The variable is stored when the player logs out and will be restored
-- when he logs back in before the addon codes are run.
-- These variables are account bound.
-- Only exists on client side and you should call it only once per key.
-- All saved data is saved to client side.
AIO.AddSavedVar(key)

-- Key is a key for a variable in the global table _G.
-- The variable is stored when the player logs out and will be restored
-- when he logs back in before the addon codes are run.
-- These variables are character bound.
-- Only exists on client side and you should call it only once per key.
-- All saved data is saved to client side.
AIO.AddSavedVarChar(key)

-- Makes the addon frame save it's position and restore it on login.
-- If char is true, the position saving is character bound, otherwise account bound.
-- Only exists on client side and you should call it only once per frame.
-- All saved data is saved to client side.
AIO.SavePosition(frame[, char])

-- AIO message class:
-- Creates and returns a new AIO message that you can append stuff to and send to
-- client or server. Example: AIO.Msg():Add("MyHandlerName", param1, param2):Send(player)
-- These messages handle all client-server communication.
msg = AIO.Msg()

-- The name is used to identify the handler function on receiving end.
-- A handler function registered with AIO.RegisterEvent(name, func)
-- will be called on receiving end with the varargs.
function msgmt:Add(name, ...)

-- Appends messages to eachother, returns self
msg = msg:Append(msg2)

-- Sends the message, returns self
-- Server side version - sends to all players passed
msg = msg:Send(player, ...)
-- Client side version - sends to server
msg = msg:Send()

-- Returns true if the message has something in it
hasmsg = msg:HasMsg()

-- Returns the message as a string
msgstr = msg:ToString()

-- Erases the so far built message and returns self
msg = msg:Clear()

-- Assembles the message string from added and appended data. Mainly for internal use.
-- Returns self
msg = msg:Assemble()
```

# Included dependencies
You do not need to get these, they are already included
- Lua serializer: https://github.com/gvx/Smallfolk
- Lua crc32: https://github.com/davidm/lua-digest-crc32lua
- Lua queue with modifications: http://www.lua.org/pil/11.4.html
- Compression for string data: https://github.com/Rochet2/lualzw/tree/zeros
- Obfuscation for addon code: http://luasrcdiet.luaforge.net/
- Sent addons' frame position saving: http://www.wowace.com/addons/libwindow-1-1/

# Special thanks
- Kenuvis < [Gate](http://www.ac-web.org/forums/showthread.php?148415-LUA-Gate-Project), [ElunaGate](https://github.com/ElunaLuaEngine/ElunaGate) >
- Laurea/alexeng/Kyromyr < https://github.com/Alexeng, https://github.com/Kyromyr>
- Foereaper < https://github.com/Foereaper >
- SaiF < https://github.com/SaiFi0102 >
- Eluna team < https://github.com/ElunaLuaEngine/Eluna#team >
- Lua contributors < http://www.lua.org/ >
