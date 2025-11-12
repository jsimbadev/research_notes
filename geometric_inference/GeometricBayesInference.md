
Bayesian inference is the technique of using statistical models and Bayes probability rule to infer parameters of the model conditioned on observation. There is a rich literature of many such techniques and algorithms.
One such family of algorithms is that of the Markov Chain Monte Carlo (MCMC) type. There has been an uptake of using geometrical properties of the target distribution, via means of derivatives in order to improve the efficiency of the algorithms with great success. 
To name some HMC and its variants, MALA, SG-MCMC.

However, this brings with it some challenges.

1. many realistic models (discrete, agent-based, simulator-based) have **non-differentiable** or intractable likelihoods;
2. even when differentiable, **gradient evaluations can be prohibitively expensive**;
3. consequently, practitioners often revert to basic random-walk or Gibbs schemes, losing geometric efficiency.

Regardless, its widely accepted the advantage geometric insight contributes to the efficiency of this family of algorithms.

(Variational Bayes also is geometry accelerated, owing to being optimization based.. can expand on this if VI is worth mentioning).

The proposal is to bring the power of geometry to wider classes of Bayesian inference techniques where the above challenges have proven to be prohibitive.

The typical setup for these modern powerful samplers puts a smoothness constraint on the model, eliminating very realistic and commonly encountered likelihoods. In these scenarios, practitioners typically fall back to more rudimentary MCMC algorithms like random walk or Gibbs Samplers.
The proposition is that even though the model itself doesn't admit smoothness in the analytic sense, intuitively most solutions imply some kind of smoothness, else "learning" and "inference" become impossible.
This insight motivates a coarse attempt at derivative information in the first approximation via finite differences. This approach is valid given any metric space which is a very general condition.

This motivates **Accepted-Direction Adaptive MCMC (AD-MCMC)**:  
a _derivative-free, geometry-aware_ sampler that learns local manifold structure **from accepted finite differences**.  
The idea is to maintain a running covariance of accepted displacements and uses it to infer statistically the geometry of the underlying manifold that the chain has empirically validated via acceptance.

The broader hope is to extend geometric ideas to the full spectrum of Bayesian inference—linking derivative-free MCMC, approximate Bayesian computation (ABC), and information-geometric formulations—so that even black-box or simulator-based models can benefit from geometric acceleration.

For instance, when I read the rejection-acceptance ABC algorithm outlined below in  [Xiahui Li et al](https://arxiv.org/pdf/2504.19698)
1. Sample parameter θ∗ from the prior distribution π(θ). 
2. Simulate data $y_{sim}$ from the model given parameter θ∗. 
3. Reduce observed yobs and simulated data $y_{sim}$ to a set of chosen summary statistics $s(y_{obs})$ and $s(y_{sim})$. 
4. Accept θ∗ if the distance $d(s(y_{obs}),s(y_{sim}))$ is less than a predefined threshold ϵ; otherwise, reject θ∗. 
5. Repeat the process until the desired number of posterior samples is obtained. The accepted parameters form an approximation to the posterior distribution

It resembled the AD-MCMC idea except that the **accepted finite differences** would be calculated in summary statistic space, and a covariance matrix here is exactly what ABC uses as a rule for acceptance, but it is static and never changing. So actually potentially we could speed up ABC by updating such a static covariance. (Numerical caveats aside)

This paper by [Tyagi et al](https://arxiv.org/pdf/1208.1065) has been a main motivator in the pursuit of leveraging statistical estimates of tangent spaces - effectively derivate information - in the ambient Euclidean manifold regime, a ubiquitous problem. It seems like it can open up derivative-free inference.
Built on PCA ultimately, If performant online algorithms can be leveraged, geometric fields like those required in analytic RMHMC and other techniques may become computationally tractable, and actually, discrete problems with implicit smoothness can possibly gain geometric acceleration.

This line of work naturally leads to the development of a global geometric mesh layer, which can be used to build a field.
I want to propose a project for the CDT software week, and possibly get some collaboration. Depending on how complex you think a proper polished version can become.

```python
McAtlas

from typing import Protocol, Any, Sequence


# Structure backed by Approximate Neareast Neighbour
# KDTree, Locality Sensitive Hash ... 
class Atlas(Protocol):
	# Semantically 'radius' defines the region of validity.
	# Not sure how this will work with scaling etc ... User should know appropriate scale?
	radius: float  
	
    def put(self, point: Sequence[float], info: Any) -> None:
        """
        Insert or update local information around a point.
        
        """
        ...

    def get_information(self, point: Sequence[float]) -> Any:
        """
        Retrieve information most relevant to this point.
        Could return exact match or interpolated aggregate.
        """
        ...

    def get_neighbors(self, point: Sequence[float], k: int = 5) -> list[tuple[Sequence[float], Any]]:
        """
        Return neighboring regions/charts near point with informatoin.
        Useful for interpolation, smoothing, or gradient estimation.
        """
        ...

    def merge(self, other: "Atlas", agg_fn=None) -> None:
        """
        Combine two atlases, optionally using agg_fn to merge overlapping information.
        Take care about merging atlases of different radii. 
        e.g. average metrics or blend covariance estimates.
        """
        ...

```

