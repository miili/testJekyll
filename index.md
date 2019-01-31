Development of BayHunter
========================

BayHunter is a tool to perform a Marcov chain Monte Carlo (McMC)
transdimensional Bayesian inversion of receiver functions (RF) and
surface wave dispersion (SWD), i.e. inverting for the velocity-depth
structure, the number of layers and noise parameters (correlation,
sigma). The self developed Bayesian inversion tool uses multiple 'Marcov
chains' to find the models with the highest likelihood, through the
random 'Monte Carlo' sampling.

Each chain contains a current model. In each iteration a new model will
be proposed, based on a proposal distribution, by modification of the
current model. The acceptance probability is computed. It includes the
prior, proposal and posterior ratios from proposal to current model. A
proposed model gets accepted with a probability equal to the acceptance
probability, i.e. if the likelihood of the proposed model is larger, it
gets accepted, but also models that are worst (= less likely) than the
current model, can get accepted with a small probability, which prevents
the chain to get stuck in a local maximum. If a proposal model got
accepted, it will replace the current model; if not, the proposal model
gets declined. This process will be repeated until the given number of
iterations is worked off. Each accepted model is collected (chain
models). The chain models after the burn-in phase define the posterior
distribution of the parameters.

The Optimizer and Chain modules
-------------------------------

The *BayHunter.mcmcOptimizer* combines the chains in an overarching
module. It starts each single chain's inversion and handles the parallel
computing. Each chain and its complete model data can still be accessed
after an inversion (in the Python terminal). Before the optimizer
initiates the chains, an automatic configfile will be saved, which is
necessary for the plotting after the inversion.

Each *BayHunter.SingleChain* explores the parameter space and proposed
models get accepted or declined by following Bayes principles. A chain
follows multiple steps, which are listed and described below in detail.
Equations given are reduced to the number of equations required in
BayHunter, but not deduced. The mathematical derivations can be looked
up in several fundamental books and papers, I am especially referring
here to Bodin et al. (2012) and the corresponding PhD thesis (Bodin,
2010).

##### Initiate the targets.

The first step towards an inversion is to define the target(s).
Therefor, the user needs to forward the observed data to the designated
*BayHunter* target type. For receiver functions assign observed
time-amplitude data to the *PReceiverFunction* (or *SReceiverFunction*)
class from *BayHunter.Targets*. You need to update the forward modeling
parameters (Gauss filter width, slowness, water level, near surface
velocity), if the default values are different from the values used for
your RF computation. For surface wave dispersion assign period-velocity
observables to the class that handles your type of surface waves
(*RayleighDispersionPhase*, *RayleighDispersionGroup*,
*LoveDispersionPhase*, *LoveDispersionGroup*). The forward modeling
parameters to update include only the mode.

Each of these target classes comes with a forward modeling code
(plugin), which is easily exchangeable. For RF the plugin is based on
RFmini from Joachim Saul, GFZ, Potsdam. For surface waves a quick
fortran-routine based on CPS-surf96 from Rob Herrmann is pre-installed.
Also other completely different targets can be defined.

##### Parametrization of the model.

The model we refer to includes the velocity-depth structure as well as
the noise scaling parameters given through the noise covariance matrix.

    **Velocity-depth structure.** The velocity-depth model is
parametrized through a variable number of Voronoi nuclei, the position
of each is given by a depth and a seismic shear wave velocity (Vs). A
model containing only one nucleus represents a half-space model, two
nuclei define a model with one layer over a half-space and so on. The
layer boundary (depth of an interface) lies equidistant between two
nuclei. The advantage to use Voronoi nuclei over a simple
thickness-velocity representation is, that one model can be parametrized
in many different ways. However, a set of Voronoi nuclei defines only
one specific model. The number of layers in a model is undefined and
will also be inverted for ("transdimensional").


$$C_e = \sigma^2R$$


The covariance matrix $C_e$ is dependent on $\sigma$ and R, which is the
symmetric diagonal-constant or Toeplitz matrix.


$$R = \begin{bmatrix}
1 & c_{1} & c_{2} & \hdots & c_{n-1}\\
c_{1} & 1 & c_{1} & \hdots & c_{n-2}\\
c_{2} & c_{1} & 1 & \hdots & c_{n-3}\\
 &  &  &  \vdots & \\
c_{n-1} & c_{n-2} & c_{n-3} & \hdots & 1
\end{bmatrix}$$

We consider two correlation laws for the correlation matrix R. The
exponential law given by $$\label{eq:exp}
c_i = r^i$$

and the Gaussian law given through

$$c_i = r^{(i^2)}$$
