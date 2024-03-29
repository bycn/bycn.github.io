# Project Nabla (Technical Blog Post)

Using a dataset of tournament players and large scale self play, we can train AIs that play Super Smash Bros. Melee in a competent and human-like manner.
<br>
<hr/>
<br>
*Note: This page describes technical details; for a more casual overview and videos, please see [this post](/2022/08/19/project-nabla.html)*

### Intro
In recent years, game-playing AIs trained using the combination of behavioral cloning and deep reinforcement learning (RL) have found tremendous success, reaching superhuman levels in difficult games such as Go and StarCraft II. This project is not the first to apply deep RL to Melee, and I encourage you to check out Vlad Firoiu's work on Philip [1](https://arxiv.org/abs/1702.06230?context=cs.AI), [2](https://arxiv.org/abs/1810.07286). The purpose of Project Nabla was to take advantage of a recent suite of Melee-specific software tools known as Slippi, which allows for (among many things) recording replays of games. This allowed us to use datasets of tournament-level players to train a behavioral cloning agent. Within the span of a few days on 1 consumer grade GPU, we can train agents that mimic human skills. By further scaling up using large-scale deep RL, agents learn emergent human-like strategies and resemble low-level human players.

### Melee as a research problem
SSBM is a 20+ year old game but it continues to have a large, thriving competitive scene known for its grassroots nature; the largest tournaments reguarly draw around 100k viewers. The main focus is on a 1 verus 1 format where characters on a platform fight each other at a fast pace. The specific details of the game are aptly summarized in [Phillip, Firoiu et al 2017](https://willwhitney.com/assets/papers/Beating.the.Worlds.Best.pdf). From a research perspective: first, the game presents a rich and complex action space (two continuous inputs and 4 independent buttons), which allows players to execute a variety of difficult maneuvers known as "tech skill" that aid gameplay. These are difficult to fully explore through dithering exploration alone (as seen in Phillip.) Players also have to make decisions under uncertainty of the opponent's actions, which has led to development of detailed "flowcharts", or set strategies that cover different options. This is further exacerbated by frame delay, which we did not address in this project but has been explored in [Firoiu et al 2018](https://arxiv.org/pdf/1810.07286.pdf). Many players express sentiment that AIs must have comparable reaction time to humans in order to be fair.
<br>
<br>
There are many other unexplored avenues using Melee as a research platform such as active learning, offline RL, preference learning, large-scale video pre-training. The suite of tools, Slippi, that allow for large dataset collection, and simple python API in libmelee, make developing for Melee much easier than comparatively popular and challenging games. Its relatively simple feature engineering and short "episodes" also allow for orders of magnitude quicker iteration than games like Dota.

### Qualitative Results
*Warning: lots of game-specific jargon ahead. For those not familiar, we recommend watching videos of the agent in action in the [casual overview.](/2022/08/19/project-nabla.html)*
<br>
<br>
Qualitatively, the behavioral cloning (BC) agent is able to learn to perform human-like skills:  wavedashing, waveshining, L cancelling, shield drops, short combos, and inconsistent recoveries. Although it learns these skills, some of which previous non-BC agents did not, it still has limitations such as temporally extended combos, coherent recoveries, edgeguarding, and avoiding silly SDs. Amusingly, with BC agents, they also learn to do "hand warmers", or displays of flashy skill during the set to keep the hands warmed up.
<br>
<br>
With additional RL fine tuning, agents are able to "stitch together" the BC skills effectively. As a baseline, the Falco BC-only agent wins (counting taking a stock as a win) less than 30% of the time against the highest level in-game CPU (hard-coded AI). After RL fine tuning, the falco agent wins more than 80% of the time. We observe emergent human-like strategies such as shield pressuring with shine and aeriels, using smash attacks at high percents to get kills, using lasers into grab, and more.
<br>
<br>
We made the BC + RL fine tuned agents available to play for anyone through a Twitch channel — reaching thousands of plays over a few weeks. Many anonymous play-testers found the on-stage play quite humanlike, with some even confused and not realizing that they were playing against a bot. The most common complaints were about its lack of edgeguarding and over-use of powerful (but punishable) smash attacks. The lack of edgeguarding was present in previous work without imitation (Phillip); we hypothesize that it is due to edgeguarding necessitating precise skills around dangerous states. (*Note: certain versions of Philip did do some edgeguarding such as offstage stomps/knees (attacks); to a lesser extent, this project also sees some offstage dairs, but the behaviors leave much to be desired.*) The smash attacks we ascribe to a combination of the agent's ability to abuse its reaction time, and inability to properly punish missed attacks like pro players do. The pro players/commentators so far who have watched the agents also find the general gameplay mostly undistinguishable from a run-of-the-mill low level player. They complimented the bot's on-stage gameplay, particularly the shield pressure, but again found off-stage play lacking compared to humans. 
<br>
<br>
Artificial limits to reaction time are commonly deployed in order to compare agents to human play more "fairly", such as in [AlphaStar](https://www.deepmind.com/blog/alphastar-grandmaster-level-in-starcraft-ii-using-multi-agent-reinforcement-learning) and [OpenAI Five](https://openai.com/five/). In this work, we do not make any significant attempt towards this problem — we only use a set 2 frame delay which is much lower than in previous work. However, we find that by using only a small amount of RL fine tuning on top of the imitation policy, our agents often continue to act as if they had a human reaction time even though they have access to privileged, superhuman information. Of course, as RL fine tuning is scaled up, it does become the case that the policies increasingly abuse their reaction time and become less humanlike. This is a trend [more generally with RL](https://openai.com/blog/faulty-reward-functions/) — when human designers try to specify a reward for agents to accomplish some goal, agents often learn to achieve reward in a way that is unwanted. This issue is closely related to problems in AI alignment; although previous work has used [human preferences for value learning](https://arxiv.org/abs/1706.03741) in order to try to learn the reward function rather than design it, future work (to the best of our knowledge) could study how using BC simultaneously with RL could enforce human intentions in the reward.

### Implementation Details
#### Observations
Like previous work, we use a low-dimensional state as input. The main features are action state, stage, character, facing, invulnerable, jumps left, on the ground, percent, shield strength, x/y position, and # stocks. When applicable, features are normalized to between -1 and 1. The observation consists of our own and the opponent's state, in addition to our last action (not the opponent's).
#### Actions
The gamecube controller is represented by an autoregressive action space. The main and c sticks are represented by their x and y position, which are each discretized into 17 parts. L/R and X/Y are collapsed into one button each, and instead of predicting whether to press each button individually, we predict two button *presses*, each of which could be one of ["A", "B', "L", "X", "Z"]. 
#### Rewards
We use the reward from [Firoiu et al](https://arxiv.org/abs/1702.06230?context=cs.AI), which is +0.01 for percent and +1 for stock, negated for opponent so that it is symmetric and zero-sum.
#### Network Architecture
We make use of embeddings for categorical features, and use dense layers for the concattenated input features. The output is then fed to a 1-layer size 256 GRU core. The core outputs are then fed to an additive autoregressive action head, with the order being button 1, button 2, main stick x, main stick y, c stick x, c stick y. During learning, teacher forcing is used for successive conditional actions.

#### Training procedure
The training procedure begins with behavioral cloning. Data preprocessing starts with a dump of ~100k slippi replays, which are filtered down to those that contain the character of interest (~20k for popular characters). We run preprocessing in parallel to parse the Slippi files (each game/replay must be parsed linearly), choose relevant feature information, and save in python objects to pickled files. To minimize file I/O time during training, we make use of Numpy memmaps. Another processing step iterates through the pickled files, performing feature engineering before storing each frame as a row in a memmap. Metadata is stored during this step so that an accompanying dataloader can efficiently sample random trajectories as well as map back to a specific game for debugging.
<br>
<br>
Agents are trained with an unroll length of 64 using a batch size of 2048 trajectories. The loss function is a standard cross entropy loss; we minimize the KL divergence between the data policy and the model's policy given the current frame and hidden state. The optimizer is Adam with a learning rate of 3e-4. The original training was done on a local "workstation"... with a 3070 GPU and 16 CPU cores. 
<br>
<br>
For reinforcement learning, we scale up to a larger scale setting using v3-8 TPUVMs generously provided by the TRC program. We use the PPO algorithm and utilize a training setup heavily based on the [Sebulba architecture](https://arxiv.org/pdf/2104.06272.pdf). For stepping environments in parallel, we adapt a Python multiprocessing based setup using shared memory to communicate observations to the main process (based on [OAI baselines](https://github.com/openai/baselines/blob/master/baselines/common/vec_env/shmem_vec_env.py)). For each step, a batch (of the size of the number of parallel environments) is evaluated on-device using 1 TPU core reserved for this purpose. Once enough steps are gathered, the batch is put on-device and then sharded across 6 learner devices, taking advantage of fast inter-device data transfer. After each set of optimization steps (using the JAX collective pmean to coordinate across learner devices), params are quickly transferred to the actor device, minimizing policy lag.
<br>
<br>
We didn't tune the RL hyperparameters and there are likely large improvements to be found here (as well as elsewhere in the architecture, algorithm, etc.) We used a batch size of 144 taking 120 steps at a time, with an unroll length of 60 and a minibatch size of 72, taking 4 gradient steps per batch. We used a 1e-4 learning rate, 0.0001 entropy coefficient, discount of 0.99 and GAE lambda of 0.95, and a PPO clip of 0.2. The value function re-used core outputs as inputs.
<br>
<br>
The parameters are initialized using the BC policy. We use RL fine tuning (with a lower learning rate) along with a KL penalty with the BC policy. We find that the weighting of the loss contribution is extremely important, although this may be algorithm-specific. To find a good balance, we tuned the weighting using a simple task (such as moving right) for the RL reward. The self-play itself is simple, with models only playing against the most recent set of parameters. We learn from the experience of both players. With certain characters, we find that policies collapse into undesirable behaviors — future work should utilize historical sampling or population-based training to alleviate similar problems.


### Conclusion
In this work we trained human-like agents for Super Smash Bros. Melee through imitation learning and fine tuning on RL self-play. Agents learn emergent human-like strategies and pass an informal "Melee Turing Test", simply through the use of a strong behavioral prior. As a game with interesting complexity combined with an active community and much smaller scale computation necessary than previous works like Dota 2 / StarCraft, we also posit Melee as an underexplored research platform, suitable for studying problems in active learning, offline RL, large-scale video pre-training, preference learning, and more.

*If you have any questions about training details or anything else, reach out to me at my email or join the Slippi Discord.*
<br>
<br>
#### Extra: Twitch queue details
Because bots are not allowed on unranked currently, we integrated with the [twitch API](https://github.com/bynect/pytmi) so any viewer can add themselves to a queue to play a direct match against the bot. The stream features a display utilizing [glitch](https://glitch.com) to host a full stack node.js web app. The app uses socket.IO to communicate between the front/back end and live-update the status of the queue. From the main process that runs the game loop, a separate thread is launched with a helper object that manages the chat bot and the website requests. The helper is shared with the main process so that game loop events can be communicated to the bot/requests and vice versa. A simple leaderboard is persisted by saving state to a file.

*Research supported with Cloud TPUs from Google's TPU Research Cloud (TRC)*

<div id="disqus_thread"></div>
<script>
    /**
    *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
    *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables    */
    /*
    var disqus_config = function () {
    this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() { // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    s.src = 'https://bycn-github-io-2.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>