# Introduction

Two Sigma has done it again. Halite 2 was three months of amazing and fun competition. While there are always aspects of the game that could be improved, I will not be focusing on that in this writeup. Halite 2 was probably a bit less accessible than Halite 1, while also being more accessible with its theme. Halite 1 will always hold a special place in my heart since it was what sucked me into competitive programming. I had been looking forward for months to see if Halite 2 was happening and I was so excited when it was. I was also given an opportunity to participate in the beta and to provide some feedback. Being able to contribute a little bit to Halite 2 made this very special as well. 

I work as an Actuary at Allstate Insurance Company. Halite 1 was my first experience with competitive programming, and my background is completely unrelated to anything related to AI or CS. It goes to show that anyone, really anyone, can learn this and perform well!

# Strategic Developments

Before going into my bot, I do want to talk a little about the "meta" shifts that occurred, or ideas that popped into / out of existance over the course of the competition. Unfortunately, since my memory fails me, these are likely not in the order that they appeared, but that doesn't really matter much.

## The Desertion Meta

We all know it, we all hate it, but we all had to do it.

I would love to claim credit for this, and I think I can, but I also want to give credit where due. I believe I was the first to implement this in the open competition. My approach for almost 80% of the competition was a very simple one. Hide a ship in the corner. Given the tiebreaker rules, as long as you were alive, you would place higher than people who were dead. Since most people used some sort of closest enemy ship metric to select targets, by hiding a ship in the corner, I'd reduce the chances that I'd be dead. Of course, this only worked for so long until everyone else started doing it.

In reality, this idea actually came from @DexGroves. My former colleague who unfortunately did not participate in the open competition, but did participate in the beta. He was the one that originally showed me this trick but quickly disabled it for the beta. I promised him I wouldn't steal it for the beta, but would use it in the open competition. It was an obvious idea so it would have just been a matter of time before someone else did it.

Hopefully we won't see the need to implement something similar in Halite 3.

## The Drone Harass

So, for those of you young'uns who don't know what the drone harass is, watch this: [Probe Rush](https://www.youtube.com/watch?v=etDGtMMhf9w)

Ok, for everyone else. I first noticed this from @ewirkerman. Early on, he would send out a lone ship (I think even as aggressively as one of his three starting ships) to harass the enemy by either targeting his docked ships, or by distracting them enough that they would start chasing it and thus lose production. Early on, most bots did not have good defense, or would overcommit to defending against enemy ships. It was a very effective strategy for a long time until people started having better enemy targeting and wouldn't have 10 ships chase a single one. You can still see this idea in many bots today, maybe not explicitly coded, but more from emergent behavior through better combat evaluation.

## Rushing

Oh rushing. How I hate you. I'm going to get the attribution wrong, but I think the first person that I saw do this was @mellendo in the beta. I noticed that he was quickly rushing to kill off people's ships early on and it was all downhill from there. I'm not going to expound upon this more since @fohristiwhirl did a great job explaining this in his writeup [Rush Theory](https://github.com/fohristiwhirl/halite2_rush_theory).

I think rushing is a broken game design that basically reduces the game to a coinflip. 

## The Prisoner's Dilemma

So rushing in 4p games was... a nuisance. Early on, when bots didn't have proper rush defense, it benefited people to rush their opponent, take them out early, and thus get half the map for yourself. However, as the meta developed, it basically became the prisoner's dilemma. If one bot rushed while the other didn't, the bot that rushed would win. If both bots rushed, then it was likely they'll kill / stall each other and the other two players who didn't rush would win. I was one of the more... enthusiastic rushers, but more out of necessity than anything else. It wasn't until the last few days where I was able to tone it down enough to still maintain/improve my win rate.

A common counter to this was that players would move to a planet, but not dock if there was the possibility of a rush occurring. See this [replay](https://halite.io/play/?game_id=7877830) for an example of the most exciting game ever. Eventually, I think it became accepted that cooperation was better than betrayal, and most people opted to dock if possible instead of rushing the enemy at the top end.

## Numeric Superiority

Sorry, I don't have a fancy name for this. I first noticed this from @zxqfl but I think @reCurs3 and @FakePsyho took this to a whole new level. Basically, it boils down to: Do not fight in a fight where you cannot win. Prior to this, most ships just attacked each other haphazardly. But, in reality, fighting for a tie does nothing. Detecting if you were outnumbered, or in a tie, meant that you should try retreating back to a friendly ship to outnumber the opponent. By the end, every top bot was doing this in some form or fashion. It's not always easy to figure out when you could attack or not. This lead to some games having a giant deathball rolling around <<REPLAY HERE>>. I can only speculate on how people implemented this but different implementations led to different behaviors.
 
## The NAP

Wait, what? Yes... The Non-Aggression Pact. Or, I guess it would be more accurately described as the [alliance](https://halite.io/play/?game_id=9266337).

@ewirkerman and I had decided to collaborate and create a NAP in 4 player games. Every game, we would send a signal in 4p games, and if the secret handshake was detected, we would not attack each other's docked ships. For all intents and purposes, our ships were invisible to each other (or... at least that was the intent). Also, we would make sure that the enemy was completely eliminated before turning on each other. This would, in theory, guarantee that we were placed 1st and 2nd when we were on the same side of the map, and in theory, would still have some benefit if we were across or diagonal from each other.

We kept it hidden until the final two hours of the competition. Surprisingly, no one noticed, and we saw a definite spike in our win rates when we were in the same game. We had done a lot of local testing to make sure we had all the bugs worked out. I even uploaded my alliance dance code for about 2 weeks, but if it detected a positive, i would immediately crash. This would give us an idea of how often our signal would have a false positive. In two weeks of playing, there were 0 false positives.

It wasn't always that easy actually. Our original signal caused me to crash in about 30% of my 4p games. However, we found out later that @ewirkerman actually had been sending his signal the entire time! Despite that, I saw games where I was crashing where he wasn't in it, so we decided to make our signal slightly more complicated so that we would reduce the frequency of false positives.

Also, I actually backstabbed him =(. What I had done is basically remove all of my allied ships from the enemy filters that I create, and the filters are what I use for any sort of decision making. Except... the ships's nearby enemy ship vectors are built off of the raw data in the game_map object... The code [here](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/base_functions.cpp?at=master&fileviewer=file-view-default#base_functions.cpp-371:405) is what creates the nearby ship vectors and I didn't add a check to the ally function since I had forgotten that's what created it. We didn't notice this until after the finals because of a bug in his code where he stays still until all planets are captured. That's when he noticed that my ships actually attack his. In our testing, we just assumed any damage done was collateral damage from ignoring the other player's ships. In reality, my ships saw that they outnumbered enemy ships on their way to their target and used that opportunity to attack. Oops.
 
# History

The competition was dominated by a few distinct eras.

## TS-Admiral

The top bot from the internal beta, @Daewook's bot. This bot held the top spot for a couple of weeks. @Vlad-Shcherbina was the first to officially beat ts-admiral to take first place on November 2nd, although @zxqfl was the first to have a stable mu above TS-Admiral. @zxqfl officially took over first place on November 4th.

## Zxqfl

The next reign was from @zxqfl. He held the top spot for a while and every time I would get close or beat him, he'd come back and beat me down to my place. I think the longest I held onto first place was for about 4 hours. His reign continued until December 3rd when @ReCurs3 took over around December 3rd.


## The ReCurs3 FakePsyho rivalry.

An arms race developed up through the midseason between @ReCurs3 and @FakePsyho. @ReCurs3 was the first to beat @zxqfl but @FakePsyho quicky caught up and surpassed @ReCurs3 on 12/11. @FakePsyho officially won the Midseason and then was fairly inactive after that. He held on to the lead until early January when @ReCurs3 took over.

## The ReCurse3 era

For the rest of the competition, @ReCurse3 dominated the top spot and no one has come close. In fact, @FakePsyho's bot held the 2nd spot for the rest of the competition as well. At this point, it's pretty much undisputed who the reigning champion of Halite 2 is.

# My Bot

I'm not going to go into too much detail about older versions of my bot. I will say that a very simple bot that prioritizes docked ships and doesn't crash will probably get you close to the top 100. A simple target loop like the following:

Find enemies within distance X from us. If no enemies are near, dock on nearest neutral/owned not full planet. Otherwise, target closest enemy or docked ship. 

Seriously. A bot as simple as that using the default starter bot and adding in rushing would probably perform very well.

One overarching theme that I recommend is. Simplify. Simplify, Simplify, Simplify.

I would say that I had three major versions of my bot. The python version, my first C++ version, and my most recent version. And, in fact, I'd probably lump the python and c++ version together into a single one. I'll talk through the structure of my final bot in this post mortem. Warning, the below is fairly detailed but walks through my logic in a better format than looking at the source code itself.

## Language Choice
Having used python in Halite 1 and running into issues with timeouts, I knew that it was time to swap to a different language for Halite 2. I experimented with Golang for Halite 1, but I couldn't get it to properly work so I abandoned it and went back to Python for Halite 1. In between Halite 1 and Halite 2, I had discovered Codingame.com and from there, I started to learn Golang. I found quickly that while Golang was faster than Python, for what I wanted to do, C++ would still be faster. The jump from Golang to C++ was much lower than the jump from Python to Golang, and I was able to quickly pick up C++.

In the beta, I did stay with Python mainly because I was familiar with it, and the starter kit for C++ was in my opinion, a complete mess. Since it was just a beta, I decided to stick with it with the intent to port over to C++ when the starter kit was fixed. Also, since I was asked to not submit my python bot for a few days, I used that opportunity to port my python bot over to C++. Given I was having some timeout issues with Python (mainly through trying to loop through things too many times), and knowing the problem scope would be orders of magnitude higher than Halite 1, I knew that I would need to swap off Python if I wanted to contend for a top place this time around.

## Starter Kit Modifications
For the most part I used the majority of the c++ starter kit but added additional properties to the bot which I explain below. Only the ones that are interesting are listed:

Entity:
* score_1, score_2 : Used for planet scoring
* vx, vy : Used to store projected velocities for collision detection
* projected_location : Used to store the projected location for this entity on the next turn
* targeted : Used to store which of my ships are targeting this entity. Important so that we don't send too many ships to a single target.

Ship:
* nav : Used to store the navigation function that we want to use for movement
* behavior : Used to store the behavior function that we want to use for target selection
* entity_target : Used to store the entity we're targeting
* nearby_enemy_ships : Stores a vector of pointers to all nearby enemy ships (used to speed up since we only care about ships that can affect us)
* nearby_friendly_ships : Stores a vector of pointers to all nearby friendly ships.
* rand1, rand2 : Random numbers generated to store some sort of persistency


## Bot Logic Overview

A turn in my bot can be broken up into the following major steps which I'll go into detail on:

1. Turn Initialization
2. Behavior/Target Assignment
3. Target Post-Processing and Reassignment
4. Navigation


### 1) Turn Initialization:

#### 1a) Simulate start of turn
One optimization that I did was to predict at the beginning of the turn, ships that would attack each other immediately and would die. If a ship would die, friendly or enemy, I would effectively ignore it for the rest of the turn. I kept track of ships that fired by setting its weapon_cooldown property to 1. I do use this, but I'm not sure if it made a significant difference.
#### 1b) Create ship vectors
To make things easier, I created vectors of ships/planets that I cared about, such as: my_ships, my_undocked_ships, enemy_docked_ships, neutral_planets, etc... This made it easy to determine what I wanted to loop through.
Each ship also has a vector of nearby friendly/enemy ships (relative to the ship). This was used because in my navigation code, I was doing a sort on ALL undocked ships for EVERY move combination, for EVERY SHIP. At least twice. C++ let me get away with a LOT of things, including horribly inefficient code. Eventually, this just got to be too much, so pruning down the entire map's worth of ships into just ships that we care about (Ships that are within 2 * MAX_SPEED + WEAPON_RADIUS + 2 * SHIP_RADIUS) made my code run significantly faster.
#### 1c) Process last turn ship data
This includes carrying over some persistent data, like the rand1 and rand2 properties of ships.
I also calculate the prior turn's enemy movements. Mainly because at one point I had primitive enemy prediction (basically, assume they move the same way they did before). This is also used for ally detection obviously.
#### 1d) Ally code check
If I had an ally in the game, I would overwrite my created ship vectors and stick them into the ally vectors.
This is also where we check if we should remain allies or if everyone else is dead.
#### 1e) Score planets
To be honest, I for the most part, did not touch this code after the first week or two. But, I assign each planet a score which is it's desireability. I actually assign 2/3 scores to planets. The first, score_1 is used for the default behavior. This score does vary between 2 player and 4 player games. score_2 is what I use for my settler behavior, which I'll talk about more later.

[Lines 493:571](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/base_functions.cpp?at=master&fileviewer=file-view-default#base_functions.cpp-493:571) in base_function.cpp contains the full detail on the primary planet scoring function, but at a high level:
* Planets that have more docking spaces are more valuable
* Planets that we own have more priority
* A penalty is given for planets with 2 docking spots at the beginning of the game
* In 2 player games, planets that are close to other planets are more valuable
* In 4 player games, planets that are closer to other neutral or owned planets are more valuable, but planets closer to the enemy are less valuable. In addition, planets that are further away from the center are more valuable (The idea is that you don't want to be flanked by multiple enemies and that we should capture our side of the map first)

[Lines 468:491](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/base_functions.cpp?at=master&fileviewer=file-view-default#base_functions.cpp-468:491) contain the full detail on the secondary planet scoring function, used for settlers
* Planets that are closer to us are more valuable
* Planets that are further from the enemy are more valuable

#### 1f) Set rush mode
This section of the code is found on [Lines 113:222](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/logic.cpp?at=master&fileviewer=file-view-default#logic.cpp-113:222) of logic.cpp
I have a global flag rush_mode which is set to true if we activate the rush mode logic of the bot. In additon, in 4p games, a rush_id property is set which identifies our rush target.

I fiddled around with the numbers here so many times it's not even funny. At the end here is what I decided to do:

For 2 player games -
If it's been less than 16 turns, the enemy has no docked ships, then look at the distance between each of our ships and each enemy ships. If it's less than 11 * MAX_SPEED away, we activate rush mode.

A last day fix that I put in was that once in a while, I might have a ship fly to the center and the other two ships fly away from the center. The enemy might do the same. In this situation, I don't actually want to activate the rush mode. So I added an additional check where we only activate rush mode if all of our ships are within 3 * MAX_SPEED from each other.

We deactivate rush mode if the enemy's closest ship is more than 11 * MAX_SPEED away from us.

For 4 player games -
This logic is a bit more complicated and gets intertwined with the actual Rush behavior which I'll discuss below.

As with the 2 player section, I changed the thresholds here so many times. At the end, I decided to go with a much less rush happy bot.

If the enemy is within 7 * MAX_SPEED of us, then we activate rush mode.

We deactivate rush mode if our rush target is dead, if the enemy's closest ship is more than 10 * MAX_SPEED from us, or we find a closer enemy that isn't our rush target.

### 2) Behavior/Target Assignment

My idea at the time I did the last refactor of my bot, was to separate the strategic decisions from the tactical decisions. I considered strategic decisions to be target selections, or basically where we should be sending ships, and then a tactical module would be used to determine the actual movement of the ship. I refer to these as behaviors and navigation functions respectively. Ships get assigned roles (behaviors) that then have potentially unique tactical modules (navigations). I started off with something like 8 different behaviors and 12 navigation functions, but over time, I simplified and collapsed them to only a few of each that I'll go in more detail below.

I did have completely separate logic for 4 player vs 2 player games but again, as time went on, I pretty much collapsed them to be fairly similar.

#### 2a) Behaviors

##### 2a-1) Settler
The settler role was probably the last one I added. One thing that I noticed, and later saw that Recurse did REALLY well, was that often times there were times that I could just send a ship off to a far off planet and the cost of that ship would pay for itself as it populated a planet that my default behavior would not have, and would not because of its target priorities.

The settler role is located on [lines 1498:1546](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-1498:1546) of behavior.cpp.

Due to a "bug", the distance that the planet is actually has almost no relevance to what planet I want to go to. Basically the settler will go to the planet with the best (lowest) score_2 value. score_2 wasn't divided by MAX_SPEED, but I add distance / MAX_SPEED to the score. When I fixed it, it seemed to perform worse so I left it as it was.

Once a planet is identified we do a quick count of nearby friendly ships vs nearby enemy ships. This is important for two reasons.

If we're alone, and there's an enemy nearby, it doesn't make sense to dock since we'll just get immediately killed. Alternatively, if we're massively outnumbered, we shouldn't dock since we're probably better off helping to defend/combat this area. The exception is that if we're trying to dock to a planet we already own, we want to reinforce it, and hopefully (I could have added an extra check I guess) there is sufficient defenses to deter the enemy from killing this ship immediately.

Otherwise, we just head toward and dock to the planet we want.

##### 2a-2) Distractor
The distractor bot was originally designed to be the "drone harass" bot. It was effective to send ships to docked ships and have it delay weaker bots from using new ships effectively and just use it to occupy 2-4 ships while trying to snipe off docked ships when possible.

As I talked about, the behaviors handle the strategic decisions, and the navigation handles the tactical function. My original navigation function for this role was the Attack_Docked_Ship_Avoid_All nav. That nav function basically always avoids getting into range of enemy undocked ships while trying to get into range of enemy docked ships. Eventually, I replaced this with what I call my dogfight nav function without changing the target selection of the distractor bot. Because of how I implemented the dogfight function, it still functions similarly to the old distractor bot behavior, but now if I have multiple distactors (or other combatants) nearby, it'll know not to just try to avoid enemy ships if we have numeric advantage. This is how I get the "swarming" behavior where I target docked ships with a subset of ships and across the map instead of a single battlefront.

The 4-player version of this is almost identical, with the exception that we don't want to target a docked ship that's too far away from us since we should be focusing on the closest enemy.

##### 2a-3) Defender
Everyone else who wasn't given a Distactor or Settler role (and it's not a rush game) gets assigned to the Defender role. In hindsight this was really a hack but whenever I tried to fix it, I end up messing up the bot so I left it as is.

[Lines 1637:1675](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-1637:1675) in behavior.cpp contains the defender code.

Basically, what we do is for every docked ship, we look outward to see if there are any enemy ships in range of that docked ship. If there is we target the friendly docked ship and assign it the defense navigation function.

Note, that this does mean at this point we are completely overdefending. Later post-processing basically does a check to see how many ships we need to defend and releases all the extra defenders to use the Default Behavior.

##### 2a-4) Default Behavior
This is the fallback behavior. If any of the other roles can't find a valid target, OR if our ship reassignment later on says that we don't need that many ships for their target, it defaults back to this role. This role is actually more important than it sounds and is actually sort of a combination of all three of the other roles at once. This was my first behavior and then I split out the other roles to have specific defined roles for certain ships that I wanted to act a certain way.

This function can be found in [lines 1009:1189](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-1009:1189) in behavior.cpp

We define an enemy range, which is basically # of turns to look out. I didn't want to get caught with a blind spot, so I actually have each ship vary by a little bit how far they look out. I figured it was better to diversify rather than to have a single static number. Not sure if it makes a huge difference, but some quick testing with various numbers showed that what I have seems to work well.

The first thing we do is get the closest enemy to this ship.

Next, we check if we are next to a planet that we can dock on. If we are, then if there are other friendly units nearby, we should dock. This was a check I added later on to encourage docking of planets in a battlezone. More often than not, any docked ships that I would lose would be "permanently" lost. This also helps against people sending ships to harass me and killing off docked ships and my original logic not being smart enough to just dock if there are other ships chasing it.

If this ship is within our enemy range, OR there are no planets for us to dock to, do the following:

* Find the closest enemy attacking ship and the closest docked enemy ship.
* If the closest enemy attacking ship is CLOSER to us than a docked enemy ship then HALF of the time, go defend and attack that ship.
* Otherwise, if we're in a 2 player game, instead, target the nearest undefended docked enemy ship if it exists.
* Else, if it's a 4 player game, or no undefended docked ships are available, just target the closest docked enemy ship.
* Lastly, if there are no docked enemy ships and no ships attacking us, then just continue with the closest enemy ship.

Note, the above logic triggers only if there exists an enemy ship within range. Just because an enemy ship is in range doesn't mean we'll actually target that ship. Instead, it's more of an indicator that it's not safe to try to dock to a planet. So, if it doesn't detect an enemy ship in range, instead, we'll try to dock to a planet. We sort the available planet by score + distance and pick the best target. I noticed that we always try to dock to the same planet and I wanted to add some variability into the equation, so I had the score_1 function weight vary depending on a random factor. So, sometimes it'll to go the most valuable planet, other times just the closest. It doesn't provide huge variation, but enough that it doesn't get caught all the time.

##### 2a-5) Rush
I had separate behaviors and navigation functions for rushing. Eventually I collapse them into part of the dogfight nav with some flags to detect rush mode in there. But my 2p rush code is simple:

[Lines 1264:1300](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-1264:1300) in behavior.cpp
Get the closest enemy ship, if a ship is docked, prioritize that target.

I did add in some extra code in to runaway if we're outnumbered but winning the damage fight. Not sure how often this triggered to be honest. But basically, if we have fewer ships but dealt more damage, just run away and hope we can end the game on time.

For four player games, it got a little more complicated. To be honest by the end of it, i'm amazed it even worked.

I had a rush_id variable set which determined who I was rushing against and all my comparisons are against that player.

Finally embracing the cooperation logic of the prisoner's dilemma, I check to see how many ships the enemy has docked and match them. If I detect that there is an enemy ship that is closing in on me (within 6 * MAX_SPEED), then I attack them. Otherwise, I move toward a planet like i normally would and sit there until they either dock, move far enough away that it's safe to dock, or I detect that they're trying to rush me.

#### 2b) Target attacking enemy ships
I added this mainly because I found that I was doing a poor job of defending against enemy attacks. This is the first loop that actually fires off and doesn't get a role assigned. Mainly because I hacked it up late to see how it worked and seemed fine.

#### 2c) Role Assignment
The rest of this loop goes through and actually assigns roles to each ship. The way I do it is basically random assignment. Basically, about 15% of ships are settler ships, 15% are distactor ships, and the rest are defenders (which basically overflow into default)

### 3) Target Post-Processing and Reassignment

#### 3a) Reassignments

So one thing that was immediately apparent was that we don't want to have 5 ships chase an enemy ship attacking us. This next part of the code is designed to handle reassigning ships that are over certain target thresholds. I've tried to fix this a bit but given the way I was handling things, it was a complete mess so I never fixed it. I don't think that the way I'm handling overflow is great but it was sufficient and would have taken more effort than I wanted to spend on it to fix.

The first thing we do is assign all starting ships to the same planet target. I wanted to keep our ships together. If a planet has only 2 docking spots, we'll still send all 3 here in this step. A later step handles overflow onto planets. We do this only for the first few turns.

We also assign only 1 (despite what the code looks like) ship to defend each planet. We loop through each of our planets, then its docked ships to see how many ships we have defending them. We keep the one that's closest to the planet, and all others then get assigned to the default behavior. While it sounds like we only have 1 ship defending each planet, because the default behavior role actually looks at enemy attacking ships, some of the released defenders will still actually end up defending the enemy, but using the default combat code instead of the defense navigation.

After that, we then cap the number of ships that we allow to target undocked ships. This prevents us from having 5 units chase a single ship. We ensure this since when we do role assignment / target selection, we add that ship to a targeted list so we loop through each enemy undocked ship and check what ships are targeting it and then reassign those ships while removing this ship from the list of undocked ships.

Lastly, we then reassign ships that are targeting beyond the docked capacity of a planet. An enhancement that I didn't do is to take into consideration ships that will spawn and immediately dock. So there is a bit of improvement here that I could have put in.

#### 3b) 4p Desertion code
There is one additional check that I use in 4p games which determine whether or not we should retreat. After our 20th ship is produced, the ship that is closest to a corner then gets reassigned to move to the corner. We do this check every turn so if a corner ship gets destroyed, we'll send another ship to the corner.

In determining if we should retreat, we basically just check if we're down to 2 planets, and if one opponent owns half the map. If so, then we retreat and just run away from the enemy. This often results in a large clump near the corner. I see other people do a rotation where they run in circles, but I figured this was sufficient from an effort perspective.

#### 3c) Anti-Rush Check
One of the last things before we actually do any navigation is to determine if we should undock any ships as a result of being rushed. This is one of the few places where I take future state into account. Basically if we can pop a ship before the enemy arrives, then we won't undock. Otherwise, if the enemy is close enough and we don't have enough undocked ships to help defend the attack, start undocking ships to help with this,

### 4) Navigation

#### 4a) Determine Moves

The most convoluted piece of the code and also the brains of the tactical engine of the bot. This is also where all the time is taken. I'll run through the logic that the navigation function does and then talk about each navigation function separately.

The first thing that I do is to sort all the undocked ships by entity id. This is important because when possible, I want to use where each ship is going in order to help guide decisions for later ships. However, since it's iterative, I don't want to use stale information. It'll make sense when I go through the navigation functions.

I actually do a minimum of 2 navigation calls, or, more accurately, determine_move() calls. I have an extra parameter which I call initial which is used to determine initial moves for all ships and then when initial == false, we are allowed to consider other ships more freely. This will be evident as I go through the code.

The actual code itself is located in [navigation.cpp](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/hlt/navigation.cpp?at=master&fileviewer=file-view-default)

I create a vector that contains the set of objects I want to ignore. I actually create two vectors, one if the kamakaze flag is set which basically means we don't ignore docked ships. This is the first place that the initial flag is used. For the first run, we ignore our own undocked ships since we do the collision check in this step as well. This way we can pick the path we want to go to, and then handle collisions afterwards.

For every undocked ship (since otherwise they can't move anyway), I go through and score every possible move with that ship's navigation function. The navigation function takes a ship's projected location to calculate a score for that ship's movement. Each move is stored with its score in a vector, which I then sort from best move to worst move. Moves that go beyond the map boundaries are not considered.

Once the list of valid moves have been scored and sorted, I then start with the best move and check against all the targets we want to avoid. The first one that we find that passes all collision checks is the one we save.

Then... we do this again. But this time with the initial parameter set to false. Now we consider our undocked ships in collision detection (along with the navigation scoring function). We run this loop up to 5 times (an arbitrarily selected number) or until we detect no collisions with any of our moves.

There are places where with a large number of ships that collision still occurs. But I decided not to worry about it, since those situations normally occur when I'm already winning so it doesn't matter.

#### 4b) Navigation Functions
Below I'll describe my navigation functions. The most complicated of which is Dogfight. We'll start with the simpler ones first.

##### 4b-1) Default_Navigation
[Lines 3:39](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-3:39) in behavior.cpp
This ultimately is only used to move a ship to a corner. A small bonus is given for moving faster, and a small bonus for navigating away from enemy ships.

##### 4b-2 & 3) Navigate_To_Planet & Navigate_To_Planet_4p_Rush
[Lines 84:165](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-84:126) in behavior.cpp
This works very similarly to Default_Navigation.
A huge bonus is given to getting into docked range. From there, we give a small bonus for moving away from enemies and a small bonus for staying away from other docked friendly ships. (Mainly to avoid traffic jams)

There's almost no difference now between the two navigation functions

##### 4b-4) Retreat_From_All
[Lines 168:204](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-168:204) in behavior.cpp

This is pretty simple as well. We look at the 10 closest enemy ships and give a bonus the further away we are from them. We heavily penalize moves that place us within potential attack range of the enemy. One side effect is that this pretty much always moves us into a corner or an edge. I never got around to penalizing this so eventually we do get cornered.

##### 4b-5) Defense
[Lines 987:1007](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-987:1007) in behavior.cpp

A simple function. The entity_target is the ship we are trying to defend. We give a bonus for staying close to the docked ship, and then we find the closest enemy ship and give a bonus for moving close to them as well. This encourages the defense ship to stay in between the docked ship we are defending and the closest enemy ship that's attacking.

##### 4b-6) Dogfight
[Lines 426:790](https://bitbucket.org/rbshum/halite-2-shummie/src/6bd0103d66feca775c2e2fffccf1c337c256722c/src/behavior.cpp?at=master&fileviewer=file-view-default#behavior.cpp-426:790) in behavior.cpp

Ok... this one is the monster, it's the most complicated piece of the entire bot and also the main brains of the whole thing. Nearly every single behavior now uses this for the tactical decision making.

This function makes extensive use of a ship's nearby_friendly_ships and nearly_enemy_ships vectors to help with performance because at one point I was sorting the entire list of friendly and enemy vectors by distance. C++ lets you get away with a LOT of things...

Also, I have a LOT of magic numbers in here that I tweaked. I'm not going to specify the exact values as they were mostly chosen through trial/error. I will merely state if we give it a bonus (relatively large/small) or a penalty. You can always refer to the code to find the exact values I used.

Here's a summary of how we evaluate moves in this function

###### 1. Move toward target
If there are no nearby enemy ships, just give a bonus for moving toward the target and return that score.

###### 2. Calculate nearby_enemy.
This is a value that estimates the enemy strength at the projected_location we want to move to.
Basically, any enemy that could possibly attack us if we move to this position increases this value by 1, any enemy which already attacked this turn is given a value of 0.5. This was used to encourage being aggressive if the enemy couldn't retaliate.

I also track whether or not there are docked enemy ships we can attack from this position.

We actually don't end up using this value for the intial move evaluation. To be explained later.

###### 3. Calculate nearby_friendly
I've played with various values here and I still don't know what's best. As with nearby_enemy, we don't actually use this value for the intial move evaluation.

Similar with nearby_enemy, we count the number of friendly ships that are within range of our projected location. This isn't perfect since there may be friendly ships in range but not in range of the enemies we are attacking. I had a hard time trying to figure out the best way to determine if we had numeric superiority and this seemed to work ok.

Friendly undocked ships give us 1 point. Ships that have fired give us half a point, and docked ships give a full point. This encourages us to be more aggressive when defending.

###### 4) Aggression
At this point we determine if we want to be aggressive, or not aggressive. A value of 1 means we want to retreat, and a value of -1 means we want to be aggressive.

If we outnumber the enemy, that is, if nearby_friendly >= nearby_enemy, then we want to attack. We also want to attack docked ships, so I put in an additional clause to be aggressive if we can attack a docked ship and we have a large number of nearby friendlies (4 or more).

In addition, for our initial move, we will always be aggressive. This will cause nearly all of our ships to try to move toward the enemy to begin with. This then lets us determine when we go through a second time to say, ok, even if we're all aggressive, we should retreat. Otherwise, if a single ship looks at a time, they'll see no one else attacking and decide no, I want to retreat and we'll never push forward.

###### 5) Rush mode handling
This part, lines 520:537, actually contains the core logic of the rush algorithm. I calculate whether or not the MOVEMENT not just the ENDING position would result in us attacking being attacked by more than 1 ship. If we're within range of 1 ship, we give it a large bonus. Otherwise, we give this move a large penalty.

I had experimented with giving a much larger bonus if we're in range of an enemy ship that's already potentially in range of another friendly ship, but scrapped it and didn't have time to fix it since it wasn't working as I wanted.

###### 6) Move to target
Give a bonus to move toward our entity target, since ultimately that's what we want to do

###### 7) Avoid Planets
I gave a slight penalty for being close to a planet. Being next to a planet restricts further movement. It's not a very large bonus, nor does the radius extend very far, but I added it in anyway.

###### 8) Aggressive scores

###### 8a) Bonus for distance traveled
A lot of the move score is used to move toward an enemy's position. However, we assume that the enemy will likely retreat if we want to attack, so I give a bonus for traveling further that's larger than the bonus for moving toward the enemy location so that it "overshoots" the target.

###### 8b) Move toward closest enemy
While we want to move toward our entity target, we also want to attack the enemy. We find the closest enemy and then give a bonus for moving toward them.

###### 8c) Stay near friendly ships
We give a bonus for staying near friendly ships. Here is where the sorting of my_undocked_ships is important. Since we sort by entity id, during the initial run, subsequent ships can use the position of earlier ships to move closer to them. We don't do have this restriction for subsequent iterations since if we want to be aggressive, we're fine using what the intial move wanted, and if we want to retreat, well we won't be in this if statement.

We give a larger bonus for being really close to a friendly ship.

###### 8d) Bonus for attacking multiple enemies
This one is a bit weird and not sure if i would still keep it, but basically, if we outnumber the enemy, we shouldn't be passive. We are ok charging in and being in range of multiple enemy ships. This section gives a bonus for doing so depending on how far we outnumber the enemy, and a penalty if we think we're going to be in range of too many enemy ships.

###### 8e) Bonus for docked enemy ships
If we can get in range to attack docked enemy ships, we give this move a fairly large bonus.

###### 8f) Aggression bonus
All else being equal, we want to be aggressive, so massively boost this move score so that non aggressive moves are used as a last resort

###### 9) Non Aggressive Scores

###### 9a) Bonus for distance traveled
We want to move further away from the enemy if we're outnumbered, so we give a bonus for being further than we are now.

###### 9b) Bonus for moving toward our target
At the end of the day, we still want to try to get to our target, so give a bonus for trying to get closer to our target.

###### 9c) Penalty for nearby_enemy_ships
For each enemy ship, we go through and give a bonus for being away from all nearby enemies.

###### 9d) Penalty for being in weapon range of an enemy
For each enemy ship that we'll end up in range of potential combat, we penalize this move. The exception is if we're in rush mode, then we give a bonus for being in range of 1 enemy ship only and a penalty otherwise.

We also give a penalty for being in weapon range of multiple enemies as we move.

###### 9e) Stay near friendly ships
We give a bonus to stay near projected locations of friendly ships that we have moved before this iteration. Likewise we give a much larger bonus for staying really close to our friends.

###### 9f) Kamakaze!
I put this in for 2 player games only. I can't remember if I tested this much for 4p games.

If there is more than 1 enemy around us and we're not at full health, then let's just crash into a docked enemy ship. My logic was that docked enemy ships are valuable, and that we should focus on killing them. If there are enemies nearby, we might die soon and better to take out a docked ship so the enemy wastes time trying to redock and lose production.

We do give this a fairly large bonus so that kamakaze moves take priority over all other non-aggressive moves.


# Thanks

Thank you first off to my loving and supportive wife. She's learned from Halite 1 that I'm basically crazy and will be MIA for a few months, yet she still supports my insanity.

Thank you to Two Sigma for an awesome competition. Halite 1 was amazing and Halite 2 was just as fun!

Thank you to the community for being so helpful. A special shoutout to @fohristiwhirl for his contributions to the community with his helpful tools. and @janzert for his beautiful statistics and general support of the competition.

Thank you to @ewirkerman for being my sounding board this time. While it doesn't look like our NAP is going to have much of an effect, it was fun to collaborate with him on this. I will say that I messed up the implementation and actually resulted in me backstabbing him :( Neither of us had caught this until the finals and it was obviously too late to change it by then.

A shoutout to my other Allstate competitors! [@CageyFeck](https://halite.io/user?user_id=1597), [@paulrobshannon](https://halite.io/user?user_id=2459), [@RuMcClu](https://halite.io/user?user_id=1436), [@necoleman](https://halite.io/user?user_id=1636), [@floorwater](https://halite.io/user?user_id=3925), [@summerjets](https://halite.io/user?user_id=1229), [@mtmelton](https://halite.io/user?user_id=3959), [@jasonmkahn](https://halite.io/user?user_id=2120), and whoever else I may have missed.
