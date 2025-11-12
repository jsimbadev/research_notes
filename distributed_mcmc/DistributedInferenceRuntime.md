
Distributed systems are a growing trend in computation in the age of big data and big compute, they offer horizontal scaling, partitioning axes and cost amortization among other benefits, but they also bring with them a potential for chaos which necessitates careful and deliberate engineering to preserve invariances, reverting from perturbed states.

Markov Chain Monte Carlo (MCMC) algorithms are sequential in nature, and by definition carry an inherent autocorrelation structure and factoring this out to enable traditional fork-join style parallelization is an open challenge, let alone distributing across NUMA architecture machines.

There are many techniques that exist that judiciously choose to fork at convenient places making the join operation more tractable. For instance, techniques that fork on chain initialization, and run MC Chains completely independently, making the join a simple concatenation. Or other techniques that partition (fork) the underlying data sets, and then use a join that is a function of weighted combinations of the partitioned posterior draws. Although computational constraints can be circumvented in this mode, the decisions to be made in the fork and join open up room for inexactness, leading to potential bias in the final aggregate posterior. Furthermore, ultimately, each partition is a treated by standard MCMC machinery, and so the same algorithmic constraints hold. 

Compute level parallelization embodies techniques like parallel tempering, where individual chains actually do different computations, and even target different distributions, constructed such that on aggregate they learn the total distribution. Hogwild Gibbs, which is a variant of the traditional Gibbs algorithm, extended to allow for asynchronous conditional kernel updates, this has been proved to break invariance in some counterexamples (Terenin).
All of the above are making a decision balancing correctness, throughput, and efficiency.

Inspiration of Hogwild and Asynchronous Gibbs and the context of a truly distributed system, we propose to model the random environment of the runtime as an exogenous controlling process whose contribution can be characterized at 3 regimes.

	i.White noise. The environment is totally random, any worker may be considered at any point in time, but no statistical relationship in time or chain state exists.
	
	ii. Weak interaction. By the theory of adaptive MCMC, weakly interacting and diminishing environments do not destroy ergodicity, which gives engineering impotus to revert back to stability as quickly as possible, for example, replacing dead workers from checkpoint within bounded time. Theory tells us that if this can be acheived, the total invariance can be preserved. 
	
	iii. State Correlated. In this mode, the environment is actively participating in the state evolution. This naturally leads to a policy driven design where diagnostics on the system and statistical level are fed back and inform sampling regions, step sizes.

Regime (i) and (ii) can be treated by a combination of mixture kernel invariance proofs, as well as diminishing adaptation convergence as per adaptive-MCMC.
Regime (iii) leads to very interesting application like the inclusion of learning policies in the MCMC loop, however the mathematical treatment for proofs of invariance much more subtle and less well understood.

We attempt to intuit modelling the runtime as above by reproducing traditional Gibbs, both sequential and random scan by introduction of a mask random variable in extended state space.

*Assumptions*

(A1) Each block conditional kernel is $\pi$-invariant
(A2) Each block is updated infinitely often 

Assumption (A1) permits a composition kernel argument for sequential Gibbs, and a mixture kernel argument for random Gibbs, where in each the mask simply selects the coordinates to be updated.

Assumption (A2) maintains ergodicity as the state space will be explored.

To model asynchrony, per iteration, model each stale coordinate as an identity kernel in the aggregate. Terenins analysis on asynchronous gives some intuition about bounded staleness, and motivates the necessity of A2, which motivates implementation to consider only kernel contributions within bounded lag, satisfying.

So, given the above assumptions, it seems possible to recover both Gibbs and its asynchronous variant within the same masked-mixture formalism. I’d like to ask whether you think this framework is sound, or if subtle dependencies might break these invariance arguments.

For example Gibbs doesn't start with invariant kernels, but still leads to invariance, however the formulation I propose kind of starts from assuming invariance. The invariance assumption naturally comes about because in a distributed implementation, an assumption I had was just not to expose sampler which had not completed burn-in.