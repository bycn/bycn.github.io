# Project Nabla: Towards human-like SSBM agents
Using a dataset of tournament players, we can train AIs that play Super Smash Bros. Melee in a competent and human-like manner.
<br>
<hr/>
<br>
*Note: This page is a high level overview; for technical details, please see [this post](/2022/08/19/project-nabla-writeup.html)*

### Training Description
Project Nabla is trained using deep neural networks. At a high level, it works in two parts: first, it uses a dataset of Slippi replays from tournament setups, learning to copy exactly what players are doing. At this point, it has no conception of what moves are "good" or "bad"; it simply tries to imitate the human players. In the second part, it plays against itself and receives rewards when it does damage/wins the stock. The agents take actions that lead to more reward, increasing their competence. By playing against itself continually, it continues to pose challenges at an appropriate level as it improves, leading to increasingly competent agents.


### Videos 
IBDW & KJH react to the bot
<iframe width="560" height="315" src="https://www.youtube.com/embed/1BCVfL1vUq4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Toph plays against 2 frame delay Falco & Fox 
<iframe
    src="https://player.twitch.tv/?video=v1566070587&parent=bycn.github.io&time=1h1m16s"
    height="400"
    width="600"
    allowfullscreen>
</iframe>

VOD of random Twitch viewers playing against it
<iframe width="560" height="315" src="https://www.youtube.com/embed/ovuP4n1o_yY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Follow [@otter_collapse](https://twitter.com/otter_collapse) for updates, perhaps will make a video :)
## FAQ

**How's this different from altf4's [SmashBot](https://www.youtube.com/watch?v=kxwPr9oxUMw) project? How's this different from Vlad Firoiu's [Philip](https://www.youtube.com/watch?v=dXJUlqBsZtE) [projects](https://www.youtube.com/watch?v=zHtgqxRxqYg)?**
altf4's project is trained by hard-coding using heuristics. As a result, it looks much less humanlike (but it's still very entertaining and not the point of their project!) Vlad's project is the most similar project; the difference is that we start off the agent with learning from Slippi replays while Vlad's starts from random actions. Note that he is also pursuing an independent project with the same goal.


**What's the dataset it was trained on? What characters did it train against?**
[Drive link](https://drive.google.com/file/d/1VqRECRNL8Zy4BFQVIHvoVGtfjz4fi9KC/view?usp=sharing) 
Includes nearly 100k SLP files from tournament setups, pruned to remove handwarmers, doubles, <30 second matches, and others. Credit to altf4 for the original dataset and Vlad Firoiu whose filtered fox dataset was used in the beginning. It's trained against all characters; Falcon was filtered out due to some (resolvable) bugs. No filtering/fine tuning using APM or MMR was used.

**Does it have delayed reaction times? Does it learn as it plays?** It is trained with two frames of delay, while a human reaction time is around 18 frames. This is an area of future work; check out Vlad Firoiu's previous work. It does not learn as it plays. It is trained once and deployed in a frozen state. (Technical note: it may have meta-learning properties as it is trained using a LSTM, but I haven't investigated this too much.)

**How can I play against it? Is the code available?** I have been hosting it on my twitch[@rakkob](https://twitch.tv/rakkob) while interest lasts — although only limited to 1 person at a time, it's reached thousands of plays over a few weeks. It features integration with the [twitch API](https://github.com/bynect/pytmi) so anybody can add themselves to a queue; a challenge to defeat the bot as fast as possible; a display utilizing glitch[https://glitch.com] to host a full stack node.js app using socket.IO to live-update the status of the queue; and a leaderboard for the challenge. The code is not going to be open sourced at this time.

**What's your next steps? How do I follow along?** I am mainly focusing on other projects currently. There is lots of interesting future work though—tackling the delayed reaction time, fine tuning on pro players, training against old opponents, etc. Reach out to me if you're interested in working on these ideas or others! Join the Slippi Discord's #artificial-intelligence channel, and follow me on twitter for updates [@otter_collapse](https://twitter.com/otter_collapse).



### Relevant Links & Credit
Thanks to: Fizzi and the Slippi team (Nikki and others) for Slippi and the FFW code that allows us to speed up training **donate to Fizzi [here](https://www.patreon.com/fizzi36)**; altf4 for libmelee and much more; Vlad Firoiu for initial dataset, headless Dolphin and related code, and various discussions, ideas, inspiration;  Krohnos for gecko code for endless time mode; Aach, Lizardy, Raffle Winner, and Toph for playtesting early versions. 

Slippi Discord (join #artificial-intelligence after getting a dev role): [https://discord.com/invite/YRzDxT5](https://discord.com/invite/YRzDxT5)

Libmelee: [https://github.com/altf4/libmelee](https://github.com/altf4/libmelee)

Vlad's libmelee and custom Dolphin w/ FFW and Null Video [https://github.com/vladfi1/libmelee/tree/dev](https://github.com/vladfi1/libmelee/tree/dev) [https://github.com/vladfi1/slippi-Ishiiruka/tree/exi-ai](https://github.com/vladfi1/slippi-Ishiiruka/tree/exi-ai)

Public SLP Database v3 (compiled by altf4): [https://drive.google.com/file/d/1VqRECRNL8Zy4BFQVIHvoVGtfjz4fi9KC/view](https://drive.google.com/file/d/1VqRECRNL8Zy4BFQVIHvoVGtfjz4fi9KC/view)

Vlad's open source imitation learning project [https://github.com/vladfi1/slippi-ai](https://github.com/vladfi1/slippi-ai)
 and RL project [https://github.com/vladfi1/phillip](https://github.com/vladfi1/phillip)

