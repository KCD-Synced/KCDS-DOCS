# API changes in 0.4.0

KCDS 0.4.0 gives the script API a new structure and new names. Old names stop working. There are no
aliases, so a resource that uses an old name fails until you update it. Change the names below, then
test your resources.

## New structure: everything is under `kcds`

SDK types are no longer globals. Each one moved to `kcds.<domain>.<Type>`, for example
`Door.spawn(...)` is now `kcds.world.Door.spawn(...)`. Standard features such as `console`,
`setTimeout` and `fetch` stay global. This applies on top of the renames below.

| Domain | Types |
| --- | --- |
| `kcds.core` | Assets, Bans, Chat, Events, Exports, Game, Http, Imports, Locales, Messages, Queue, Runtime, SharedState |
| `kcds.math` | Color, Quaternion, Vector2, Vector3, Vector4 |
| `kcds.players` | Appearances, Buffs, DerivedPlayerStat, Entity, Interactions, Inventory, Items, LocalPlayer, Needs, Pickpocketing, Player, PlayerStat, Progression, Trade |
| `kcds.characters` | Animations, Dialogue, Dog, Follow, Horse, HorseLooks, LocalNpc, Npc, PatrolRoute, Quest, Vendor |
| `kcds.world` | Area, Audio, Blip, Cart, CombatMarker, CookPot, Crafting, Dice, Door, Gate, Gathering, Gizmo, Gizmos, Grave, GroundItem, LevelEdit, Light, Marker, Prop, PropGhost, PropPlacer, Recipe, Render, RestrictedAreaMarker, SiegeEngine, SiegeLadder, SmartObject, Stash, StonePile, Travel, Trees, Vfx, World, WorldMap, WorldResource |
| `kcds.ui` | ActionHint, Book, Camera, CharacterCreator, Compass, Controls, Discord, EmoteWheel, Freecam, Hud, Key, Keybinds, LoadingScreen, Nameplates, NativeClip, NativeElement, NativeScreen, NativeUI, NoClip, ThirdPerson, Web |

## Web views

| Old | New |
| --- | --- |
| `fw://resources/<resource>/<path>` | `kcds://resources/<resource>/<path>` |
| `window.callEvent(name, json)` (in the page) | `window.sendToGame(name, json)` |
| `Web.createView(url, options)` | `Web.open(url, options)` |
| `Web.destroyView(view)` | `Web.close(view)` |
| `Web.showView(view)` | `Web.show(view)` |
| `Web.hideView(view)` | `Web.hide(view)` |
| `Web.focusView(view, focused)` | `Web.focus(view, focused)` |
| `Web.isViewVisible(view)` | `Web.isVisible(view)` |
| `Web.setViewOffscreen(view, offscreen)` | `Web.setBackground(view, background)` |
| `Web.isViewOffscreen(view)` | `Web.isBackground(view)` |
| `Web.loadURL(view, url)` | `Web.navigate(view, url)` |
| `Web.resizeView(view, width, height)` | `Web.resize(view, width, height)` |
| `Web.setViewPosition(view, x, y)` | `Web.move(view, x, y)` |
| `Web.getScreenSize()` | `Web.screenSize()` |

## Web view events

| Old | New |
| --- | --- |
| `browserCreated` | `webViewOpened` |
| `browserLoadingStart` | `webViewLoading` |
| `browserDocumentReady` | `webViewReady` |
| `browserLoadingFailed` | `webViewFailed` |
| `browserNavigate` | `webViewNavigated` |
| `browserPopup` | `webViewPopup` |
| `browserCursorChange` | `webViewCursor` |
| `browserTooltip` | `webViewTooltip` |
| `browserInputFocusChange` | `webViewFocus` |
| `browserResourceBlocked` | `webViewBlocked` |
| `browserConsoleMessage` | `webViewConsole` |
| `browserOriginChange` | `webViewOrigin` |

## Events

| Old | New |
| --- | --- |
| `Events.emitServer(...)` | `Events.toServer(...)` |
| `Events.emitAllClients(...)` | `Events.broadcast(...)` |
| `Events.emitTo(...)` | `Events.toResource(...)` |
| `Events.emitLocal(...)` | `Events.fire(...)` |
| `Events.onLocal(...)` | `Events.listen(...)` |
| `Events.onClient(...)` | `Events.fromClient(...)` |
| `Events.onceClient(...)` | `Events.onceFromClient(...)` |
| `Events.offClient(...)` | `Events.offFromClient(...)` |

## Players, NPCs and the world

| Old | New |
| --- | --- |
| `Nametags` (client) | `Nameplates` |
| every `...Nametag...` method, such as `setNametagColor`, `setNametagText`, `isNametagVisible` | the same with `Nameplate`: `setNameplateColor`, `setNameplateText`, `isNameplateVisible` |
| `npc.nametag` | `npc.nameplate` |
| `virtualWorld` (property, option, event field) | `dimension` |
| `setVirtualWorld(...)` | `setDimension(...)` |
| `setVisibleTo(...)` | `showOnlyTo(...)` |
| `player.hardwareId`, `connection.hardwareId` | `player.deviceId`, `connection.deviceId` |

`getIdentifier('hardware')` keeps its name, so bans already saved on your server keep working.

## Scripting core and package.json

| Old | New |
| --- | --- |
| `ExecutionEnvironment` | `Runtime` |
| `ExecutionEnvironment.frameworkVersion` | `Runtime.apiVersion` |
| `ExecutionEnvironment.modVersion` | `Runtime.serverVersion` |
| `StateBag` | `SharedState` |
| `kcds.resourceDependencies` in `package.json` | `kcds.requires` |

## Error messages

If a script checks the text of an error, update it:

| Old | New |
| --- | --- |
| `Target resource not found` | `Resource is not available` |
| `No handler for message type` | `Resource does not handle this message type` |
| `Chat color must be a Color or a packed 0xRRGGBBAA number` | `Chat color must be a Color, or a number packed as 0xRRGGBBAA` |

Errors that said `virtual world` now say `dimension`.

## More renamed members

These members changed too, whichever type they are on. Old names stop working.

| Old | New |
| --- | --- |
| `addScreenEffect` | `startEffect` |
| `animalSpawnpoints` | `animalSpots` |
| `blockedPerks` | `lockedPerks` |
| `blockPerks` | `lockPerks` |
| `burstOn` | `burstFrom` |
| `canAct` | `ableToAct` |
| `cancelCombat` | `stopFight` |
| `canHit` | `inReach` |
| `carriedBy` | `heldBy` |
| `carriedItem` | `heldItem` |
| `carryItem` | `holdItem` |
| `clearShot` | `endShot` |
| `closeDuration` | `closingTime` |
| `collisionCeiling` | `wallTop` |
| `collisionFloor` | `wallBottom` |
| `collisionThickness` | `wallThickness` |
| `combatZone` | `fightZone` |
| `containsPlayer` | `holdsEntity` |
| `createVirtual` | `createStorage` |
| `crewPlaces` | `crewSeats` |
| `crewRequired` | `crewMinimum` |
| `damageRadius` | `blastRadius` |
| `destroyWithOwner` | `removedWithOwner` |
| `disableAction` | `blockAction` |
| `dismountRider` | `unseatRider` |
| `dropInventory` | `dropAll` |
| `droppedBy` | `leftBy` |
| `enableAction` | `unblockAction` |
| `entitiesInRadius` | `entitiesAround` |
| `equipGear` | `fitGear` |
| `findPerk` | `perkNamed` |
| `fistsUp` | `guardUp` |
| `gearItems` | `gearList` |
| `gearLocked` | `gearFixed` |
| `gearPresets` | `gearSets` |
| `getAimGuide` | `aimLine` |
| `getAnimalLoot` | `lootOf` |
| `getAtPlayer` | `aroundEntity` |
| `getDerivedStat` | `derivedStat` |
| `getForArea` | `linkedTo` |
| `getGroundZ` | `groundHeight` |
| `getHerbs` | `herbsWithin` |
| `getItemPrice` | `itemPrice` |
| `getItemStats` | `itemStats` |
| `getLastError` | `lastFailure` |
| `getOccupant` | `whoSits` |
| `getOpenBook` | `currentBook` |
| `getPurse` | `purseOf` |
| `getSlotCount` | `slotTotal` |
| `getTrack` | `trackOf` |
| `hasTrigger` | `triggerExists` |
| `healthyStamina` | `staminaCap` |
| `hideCodexActionHint` | `hideCodexHint` |
| `hideCurrentTutorial` | `closeTutorial` |
| `hideInfoText` | `hideBanner` |
| `holderId` | `heldById` |
| `isCategoryVisible` | `mapCategoryShown` |
| `isDefaultRelay` | `relaysChat` |
| `isElementVisible` | `hudPartShown` |
| `isInDialogue` | `inConversation` |
| `isMonologueEnabled` | `innerVoiceOn` |
| `isScreenEffectActive` | `effectRunning` |
| `isShotActive` | `inShot` |
| `isStoryCutscenesEnabled` | `cutscenesOn` |
| `isVirtual` | `isStorage` |
| `keyItem` | `lockKey` |
| `levelGuid` | `placedGuid` |
| `loadDuration` | `reloadTime` |
| `lockDifficulty` | `lockLevel` |
| `lockedByServer` | `serverLocked` |
| `lockFanciness` | `lockStyle` |
| `lockpickable` | `canBePicked` |
| `lookDirection` | `gaze` |
| `markCount` | `markerTotal` |
| `matchOf` | `gameOf` |
| `modelGroup` | `meshGroup` |
| `modelName` | `meshName` |
| `moveDirTag` | `moveDirection` |
| `moveSpeedTag` | `movePace` |
| `moveWaypoint` | `shiftWaypoint` |
| `nativeXp` | `gameXp` |
| `navigationStatus` | `travelStatus` |
| `offUse` | `stopWhenUsed` |
| `onUse` | `whenUsed` |
| `openDuration` | `openingTime` |
| `outfitPresets` | `outfitSets` |
| `patrolState` | `routeState` |
| `playOnEntity` | `playFrom` |
| `pointsOfInterest` | `landmarks` |
| `previousWeather` | `lastWeather` |
| `projectToScreen` | `toScreen` |
| `pullDownRider` | `dragRider` |
| `putDownItem` | `dropHeldItem` |
| `putPlayer` | `seatPlayer` |
| `queryNearest` | `closestTo` |
| `rainAmount` | `rainLevel` |
| `rainIntensity` | `rainStrength` |
| `raycastAll` | `traceAll` |
| `rayThroughScreen` | `screenLine` |
| `rearAndThrowDown` | `buckRider` |
| `refillFromInventory` | `topUpFrom` |
| `registerRecipe` | `defineRecipe` |
| `relativeSkills` | `scaledSkills` |
| `relativeStats` | `scaledStats` |
| `removeAllScreenEffects` | `endAllEffects` |
| `removeGear` | `takeOffGear` |
| `removeHerbs` | `clearHerbs` |
| `removePlayer` | `unseatPlayer` |
| `removeScreenEffect` | `endEffect` |
| `replicasInRadius` | `replicasAround` |
| `requestHide` | `askHide` |
| `resetRespawnTime` | `resetRegrowth` |
| `resolveGround` | `snapToGround` |
| `respawnAll` | `regrowEverywhere` |
| `respawnHerbs` | `regrowHerbs` |
| `respawnTime` | `regrowTime` |
| `respecPerks` | `resetPerks` |
| `restoreProgression` | `loadProgression` |
| `revealAllFog` | `clearAllFog` |
| `revealFog` | `clearFog` |
| `screenEffects` | `effectList` |
| `screenRay` | `rayFromScreen` |
| `sessionOf` | `activeSession` |
| `setAimGuide` | `showAimLine` |
| `setBuyPrices` | `setOffers` |
| `setCategoryVisible` | `setMapCategory` |
| `setDefaultRelay` | `setChatRelay` |
| `setElementVisible` | `setHudPart` |
| `setEntityParameter` | `setSoundParameter` |
| `setEntitySwitch` | `setSoundSwitch` |
| `setFacialExpression` | `setExpression` |
| `setFireLit` | `setFire` |
| `setGear` | `setTack` |
| `setItemName` | `renameItem` |
| `setItemStats` | `overrideStats` |
| `setKnowledge` | `grantKnowledge` |
| `setLegibility` | `setReadable` |
| `setLevelCap` | `setMaxLevel` |
| `setMonologueEnabled` | `setInnerVoice` |
| `setMovementMode` | `setPace` |
| `setNativeXp` | `setGameXp` |
| `setOperator` | `assignGunner` |
| `setProperties` | `editRow` |
| `setPurse` | `fillPurse` |
| `setRespawnTime` | `setRegrowth` |
| `setShot` | `startShot` |
| `setSlot` | `assignSlot` |
| `setSlots` | `assignSlots` |
| `setStock` | `setShelf` |
| `setStoryCutscenesEnabled` | `setCutscenes` |
| `setTalking` | `setSpeaking` |
| `setTargetable` | `setLockable` |
| `setVeto` | `setRefusal` |
| `setXpBudget` | `setXpCap` |
| `setXpRate` | `setXpGain` |
| `showComboLearned` | `notifyCombo` |
| `showInfoText` | `showBanner` |
| `showPerkGained` | `notifyPerk` |
| `showPerkUsed` | `notifyPerkUse` |
| `showRandomEventResult` | `notifyEventResult` |
| `showReputationChanged` | `notifyReputation` |
| `showXpGain` | `notifyXp` |
| `stanceTag` | `bodyStance` |
| `stationId` | `stationKey` |
| `stopAllOnEntity` | `silenceEntity` |
| `stopOnEntity` | `stopFrom` |
| `unblockPerks` | `unlockPerks` |
| `updateCustom` | `editCustom` |
| `updateScreenEffect` | `changeEffect` |
| `vendorOf` | `shopOf` |
| `visorClosed` | `visorDown` |
| `weatherBlending` | `weatherFade` |
| `weatherRemaining` | `weatherLeft` |
| `withLabel` | `taggedWith` |
| `xpForLevel` | `xpNeeded` |
