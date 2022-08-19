# Project Nabla: Towards human-like SSBM agents
Using a dataset of tournament players, we can train AIs that can play Super Smash Bros. Melee in a competent and human-like manner.
<br>
<hr/>
<br>
*Note: this is a work in progress. The beginning is catered towards more casual readers and ends with a more technical writeup.*

### General 
Project Nabla trains deep neural networks using behavioral cloning and reinforcement learning. At a high level, that means it works in two parts: first, it uses a dataset of Slippi replays from tournament setups, learning to copy exactly what players are doing. At this point, it has no conception of what moves are "good" or "bad"; it simply tries to imitate the human players. To have it learn, it then plays against itself and receives rewards when it does damage/wins the stock. This reawrd propagates back to encourage actions that maximize the reward, leading to a more competent agent.

### Videos 
<iframe
    src="https://player.twitch.tv/?video=v1566070587&parent=bycn.github.io"
    height="400"
    width="600"
    allowfullscreen>
</iframe>
## FAQ


#### Relevant Links


### Intro
In recent years, game-playing AIs trained using deep reinforcement learning (RL) have found tremendous success, reaching superhuman levels in difficult games such as Go and StarCraft II. Indeed, this project is not the first to apply deep RL to Melee, and I encourage you to check out Vlad Firoui's work on Philip.

...Work in progress :)