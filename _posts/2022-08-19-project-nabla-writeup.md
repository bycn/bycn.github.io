# Project Nabla (Technical Details)

Using a dataset of tournament players and large scale self play, we can train AIs that play Super Smash Bros. Melee in a competent and human-like manner.
<br>
<hr/>
<br>
*Note: This page describes technical details; for a more casual overview and videos, please see [this post](/2022/08/19/project-nabla.html)*

### Intro
In recent years, game-playing AIs trained using the combination of behavioral cloning and deep reinforcement learning (RL) have found tremendous success, reaching superhuman levels in difficult games such as Go and StarCraft II. This project is not the first to apply deep RL to Melee, and I encourage you to check out Vlad Firoui's work on Philip (see the related work section at the end.) The purpose of Project Nabla was to take advantage of a recent suite of Melee-specific software tools known as Slippi, which allows for (among many things) recording replays of games. This allowed us to use datasets of tournament-level players to train a behavioral cloning (+RL) agent. Within the span of a few days on 1 consumer grade GPU, we can train agents that play like low-level human players. The end product is the result of further scaling up using larger-scale deep RL.

### Melee as a research problem
SSBM is a 20+ year old game but it continues to have a large, thriving competitive scene known for its grassroots nature; the largest tournaments reguarly draw around 100k viewers. The main focus is on a 1 verus 1 format where characters on a platform fight each other at a fast pace. The specific details of the game are aptly summarized in [Phillip, Firoui et al 2017](https://willwhitney.com/assets/papers/Beating.the.Worlds.Best.pdf). From a research perspective: first, the game presents a rich and complex action space (two continuous inputs and 4 independent buttons), which allows players to execute a variety of difficult maneuvers known as "tech skill" that aid gameplay. These are difficult to fully explore through dithering exploration alone (as seen in Phillip.) Players also have to make decisions under uncertainty of the opponent's actions, which has led to development of detailed "flowcharts", or set strategies that cover different options. This is further exacerbated by frame delay, which we do not address in this project but is explored in [Firoui et al 2018](https://arxiv.org/pdf/1810.07286.pdf). Many players express that AIs must have comparable reaction time to humans in order to be fair. In our work, we find that although some players find a few scenarios unfair, overall they wouldn't suspect that our agents were bots if not told previously. 
<br>
<br>
There are many other unexplored avenues using Melee as a research platform such as active learning, offline RL, preference learning, large-scale video pre-training. The suite of tools, Slippi, that allow for large dataset collection, and simple python API in libmelee, make developing for Melee much easier than comparatively popular and challenging games. Its relatively simple feature engineering and short "episodes" also allow for orders of magnitude quicker iteration than games like Dota.

### Qualitative Results
It's best to watch videos of the agent in action! [Casual overview.](/2022/08/19/project-nabla.html)  Nevertheless, qualitatively, the behavioral cloning agent is able to learn to perform human-like skills such as wavedashing, waveshining, L cancelling, shield drops, short combos, and (inconsistent) recoveries. Although it learns these skills, many of which previous non-BC agents did not, it still has limitations such as temporally extended combos, coherent recoveries, and avoiding silly SDs which humans wouldn't make (self destructs). Amusingly, with behavioral cloning agents, they also learn to do "hand warmers", or displays of flashy skill during the set to keep the hands warmed up.

### Implementation Details

### Lessons Learned

### Related and Future Work



