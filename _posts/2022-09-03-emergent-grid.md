## Hide and Seek in a Gridworld
<video width="320" height="240" controls>
  <source src="/images/squares.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>
<br>
*Pictured: the yellow and red squares are playing a modified "hide and seek", where the goal is for the yellow agent to reach the green square. While the seeker (yellow) counts down in the first half of the game, the "hider" (red) gets to hide the green square by using the brown "boxes" to block the seeker's path. This complex behavior emerges through a simple zero-sum reward whenever the seeker reaches the green square.*

A high level reflection of my journey replicating a large-scale RL result by using a tiny-scale environment + fully on-device RL with JAX. 
<br>
<hr/>
<br>
A few years ago in 2019, when OpenAI was pursuing more "pure simulation" type projects, one of their most striking results was the Hide and Seek project, which was popular to a wide audience beyond the machine learning community. The research belongs to a vein that focuses on the "problem problem", or designing environments and game rules that are conducive to emergent complexity, rather than focusing on solving existing benchmarks. They wanted to push the limits of self-play by asking what emergent behavior might arise by combining: a simple physics-based environment, with a simple team game; with a simple algorithm; with a simpl...y massive amount of compute. In other words: what happens if you play hide and seek with some boxes, start from flopping around randomly, and learn over billions of actions? Oh, and the only reward is based on if the hider has been found :) It turns out that they learned quite human-interpretable strategies like hiders creating a "shelter" to block out seekers, and seekers using "ramp" items to propel themselves into the shelter (in addition to entertaining physics exploits... read more in their [blogpost](https://openai.com/blog/emergent-tool-use/)!). 
<br>
<br>
Rather than just focusing on the exorbitant amount of compute and dismissing the result, I found it surprising that it worked at all. Indeed, even though large-scale RL is quite [unconvincing](https://twitter.com/unixpickle/status/1543345365415391232) compared to how humans solve problems, techniques that are able to "ride the compute wave" may be enough to solve useful problems in the futur anyways. From a research perspective, this project was an attempt to verify the strength of large-scale RL for hard tasks firsthand, without any extensive tricks. If it failed, I knew the code would be useful towards future projects anyways (and indeed, much of it was copy pasted for another project).
<br>
<br>
Of course, I didn't have access to an OpenAI-sized compute cluster, nor am I a bitcoin billionaire, but I did have a local workstation and access to v3-8 TPUVMs generously provided by the TRC program. Spurred on by recent progress in hardware-accelerated simulators such as Google's Brax for TPUs and NVIDIA's Isaac-Gym for GPUs, I first considered re-creating hide and seek using one of these projects. I quickly learned that these (as of March 2022) were in their infancy, with Brax in particular having poor contact physics along with other Jax annoyances, and Isaac Gym being pretty closed off with little support for custom environments. (Note that I find these efforts very promising and they've already led to [some papers](https://arxiv.org/pdf/2206.08686.pdf); also, some multi-agent environments were already being worked on internally when I was working on this project.) After I was pointed to [this DeepMind paper](https://arxiv.org/abs/2104.06272), I decided to write my own simple gridworld environment and base my RL code off of Brax's provided JAX PPO [code](https://github.com/google/brax/tree/main/brax/training/agents/ppo). (I ended up rewriting most of it in addition to adding things such as recurrent policies, but it provided a nice starting point.)
<br>
<br>
Learning JAX was definitely a difficult transition after coming from writing PyTorch code, and there was  a significant downgrade in my iteration speed. However, after overcoming the initial roadblocks, I do prefer the functional style. Debugging jitted code is still a huge pain, profiling is hard (without what I hear are a great suite of internal-only tools...), and the functional style requires more complex thinking/planning, but what you get in control over optimization (and being able to run on TPUs, although PyTorch-XLA can be usable) can be worth the tradeoff. In my case, by writing an environment that was simulated fully using on-device observations, the entire RL loop could be jitted, leading to extremely fast simulations.
<br>
<br>
When designing the environment, I had the goal in mind that the agents should have to display the same "tool use" (note: it's debatable how significant this is as a result) as the original paper, but in a setting as simple as possible. I decided on a grid of 8x8 pixels, and instead of having it be a standard hide and seek game, the seeker's goal is to reach a target square instead. This way, pixels that represent "boxes" only need to be pushed in order to hide the goal, rather than requiring hiders to learn to pull boxes in order to properly hide themselves. For the first half of the episode, the seeker is frozen for the "preparation phase" as the hider gets a head start to push the boxes around.  Similar to the paper, we use a zero-sum reward of +1 whenever the seeker reaches the green square, and -1 if not; the reward is negated for the hider. There is 0 reward during the preparation phase. 
<br>
<br>
The self-play is straightforward (no historical sampling, no populations) and both agents share the same network. Throughout training, there are some milestones: first, the seeker learns the simple reacher task quickly, reaching the goal without interference by the hider. Then, the hider learns to use itself to block the path to the goal, putting itself in the way to obstruct the seeker. Finally, the hider slowly learns to use the boxes and itself to surround the goal. Amusingly, it chooses to use itself as a blocker even if a block is available and it has ample time - and indeed, there is no difference in reward for either choice so it is not incentivized to pursue a particular one. Similarly, it often opens and closes paths to the goal, only forcefully closing them when the seeker gets close.
<br>
<br>
Although I conceptually expected this to work, I still found myself somewhat surprised at the results. This was due to the difficulty of the problem, which although is aided by the natural curriculum of self-play, is still quite sparse for the hider to learn. It really is necessary to use extremely large batch sizes to overcome a weak gradient signal; but more than that, the larger scale also papers over a lot of the well-documented instabilities of RL. Once everything was set up properly, the first run worked — no tuning of hyperparameters or architectures necessary other than choosing reasonable defaults.

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