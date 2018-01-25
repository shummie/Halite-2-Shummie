## Introduction

Two Sigma has done it again. Halite 2 was three months of amazing and fun competition. While there are always aspects of the game that could be improved, I will not be focusing on that in this writeup. Halite 2 was probably a bit less accessible than Halite 1, while also being more accessible with its theme. Halite 1 will always hold a special place in my heart since it was what sucked me into competitive programming. I had been looking forward for months to see if Halite 2 was happening and I was so excited when it was. I was also given an opportunity to participate in the beta and to provide some feedback. Being able to contribute a little bit to Halite 2 made this very special as well. 

## Language Choice
Having used python in Halite 1 and running into issues with timeouts, I knew that it was time to swap to a different language for Halite 2. I experimented with Golang for Halite 1, but I couldn't get it to properly work so I abandoned it and went back to Python for Halite 1. In between Halite 1 and Halite 2, I had discovered Codingame.com and from there, I started to learn Golang. I found quickly that while Golang was faster than Python, for what I wanted to do, C++ would still be faster. The jump from Golang to C++ was much lower than the jump from Python to Golang, and I was able to quickly pick up C++.

In the beta, I did stay with Python mainly because I was familiar with it, and the starter kit for C++ was in my opinion, a complete mess. Since it was just a beta, I decided to stick with it with the intent to port over to C++ when the starter kit was fixed. Also, since I was asked to not submit my python bot for a few days, I used that opportunity to port my python bot over to C++. Given I was having some timeout issues with Python (mainly through trying to loop through things too many times), and knowing the problem scope would be orders of magnitude higher than Halite 1, I knew that I would need to swap off Python if I wanted to contend for a top place this time around.

## Strategic Developments

Before going into my bot, I do want to talk a little about the "meta" shifts that occurred, or ideas that popped into / out of existance over the course of the competition. Unfortunately, since my memory fails me, these are likely not in the order that they appeared, but that doesn't really matter much.

###The Desertion Meta

We all know it, we all hate it, but we all had to do it.

I would love to claim credit for this, and I think I can, but I also want to give credit where due. I believe I was the first to implement this in the open competition. My approach for almost 80% of the competition was a very simple one. Hide a ship in the corner. Given the tiebreaker rules, as long as you were alive, you would place higher than people who were dead. Since most people used some sort of closest enemy ship metric to select targets, by hiding a ship in the corner, I'd reduce the chances that I'd be dead. Of course, this only worked for so long until everyone else started doing it.

In reality, this idea actually came from @DexGroves. My former colleague who unfortunately did not participate in the open competition, but did participate in the beta. He was the one that originally showed me this trick but quickly disabled it for the beta. I promised him I wouldn't steal it for the beta, but would use it in the open competition. It was an obvious idea so it would have just been a matter of time before someone else did it.

Hopefully we won't see something similar in Halite 3.

### The Drone Harass

So, for those of you young'uns who don't know what the drone harass is, watch this: <INSERT LINK>

Ok, for everyone else. I first noticed this from @ewirkerman. Early on, he would send out a lone ship (I think even as aggressively as one of his three starting ships) to harass the enemy by either targeting his docked ships, or by distracting them enough that they would start chasing it and thus lose production. Early on, most bots did not have good defense, or would overcommit to defending against enemy ships. It was a very effective strategy for a long time until people started having better enemy targeting and wouldn't have 10 ships chase a single one. You can still see this idea in many bots today, maybe not explicitly coded, but more from emergent behavior through better combat evaluation.

### Rushing

Oh rushing. How I hate you. I'm going to get the attribution wrong, but I think the first person that I saw do this was @mellendo in the beta. I noticed that he was quickly rushing to kill off people's ships early on and it was all downhill from there. I'm not going to expound upon this more since @fohristiwhirl did a great job explaining this in his writeup <LINK HERE>.

I think rushing is a broken mechanic that basically reduces the game to a coinflip.

### The Prisoner's Dilemma

So rushing in 4p games was... a nuisance. Early on, when bots didn't have proper rush defense, it benefited people to rush their opponent, take them out early, and thus get half the map for yourself. However, as the meta developed, it basically became the prisoner's dilemma. If one bot rushed while the other didn't, the bot that rushed would win. If both bots rushed, then it was likely they'll kill / stall each other and the other two players who didn't rush would win. I was one of the more... enthusiastic rushers, but more out of necessity than anything else. It wasn't until the last few days where I was able to tone it down enough to still maintain/improve my win rate.

A common counter to this was that players would move to a planet, but not dock if there was the possibility of a rush occurring. See this game (https://halite.io/play/?game_id=7877830) for an example of the most exciting game ever. Eventually, I think it became accepted that cooperation was better than betrayal, and most people opted to dock if possible instead of rushing the enemy at the top end.

### Numeric Superiority

Sorry, I don't have a fancy name for this. I first noticed this from @zxqfl but I think Recurs3 and FakePsyho took this to a whole new level. Basically, it boils down to: Do not fight in a fight where you cannot win. Prior to this, most ships just attacked each other haphazardly. But, in reality, fighting for a tie does nothing. Detecting if you were outnumbered, or in a tie, meant that you should try retreating back to a friendly ship to outnumber the opponent. By the end, every top bot was doing this in some form or fashion. It's not always easy to figure out when you could attack or not. This lead to some games having a giant deathball rolling around <<REPLAY HERE>>. I can only speculate on how people implemented this but different implementations led to different behaviors.
