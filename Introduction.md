# Abstract #
GPGMP provides an implementation of a spatially-resolved stochastic simulation algorithm on general-purpose graphics processing units (GPGPUs). This algorithm, designed to solve reaction-diffusion equations on a mesoscopic level, has widespread applications in computational biology, molecular chemistry and social systems research. For our implementation, we use the Gillespie multi-particle method, which is well-suited for GPGPUs. In preliminary tests, we could obtain speed ups of around 1-2 orders of magnitudes as compared to standard implementations on CPUs.

# Reaction-Diffusion Networks #
[Reaction-diffusion networks](http://www.scholarpedia.org/article/Reaction-diffusion_systems) are systems consisting of one or more interacting entities (e.g. molecules in a solution) which propagate in space through random walks. _Macroscopically_, these systems obey a reaction-diffusion equation of the form
<wiki:gadget url="http://mathml-gadget.googlecode.com/svn/trunk/mathml-gadget.xml" border="0" up\_content="(del x\_i)/(del t)=f\_i(bb{x}) + D\_i grad^2 x\_i,"/>

where **x** denotes the macroscopic state vector (e.g. the chemical concentration of each species), **f** describes the interactions and **D** are the diffusion constants for each species.

In many [applications](#Applications.md), most notably in a biological and chemical context, the number of participating agents is small and the inherently stochastic nature of the problem requires a _microscopic_ description of the system, in which the position of every species is known at all times. The macroscopic diffusion equation can be regained by appropriate statistic averaging.

A microscopic approach, however, is often cumbersome and computer simulations to obtain realizations of reaction-diffusion networks are quite costly. Furthermore, it is often not necessary to distinguish between individual entities and a _mesoscopic_ approach is sufficient. In a mesoscopic description, the domain of interest is divided into small subvolumes of a convenient granularity, which are assumed to be perfectly mixed. Instead of tracing the position of each individual particle through time, one then collectively keeps track of the number of particles in each subvolume.

# Gillespie Multiparticle Algorithm #
A wide variety of stochastic simulation algorithms exist to compute single realizations of reaction diffusion networks `[`see, e.g., [Vigelius et al. (in preparation)](References#Vigelius10.md) for a brief overview`]`. Gillespie's direct method [(Gillespie, 1977)](References#Gillespie77.md) has emerged as a standard method to simulate spatially _homogeneous_ reaction networks. Very efficient adaptations of this algorithm exist, such as the next-reaction method [(Gibson et al., 2000)](References#Gibson00.md) or the logarithmic direct method [(Li et al., 2006)](References#Li06.md). Based on the first reaction method is a prominent algorithm to simulate spatially _inhomogeneous_ reaction-diffusion networks, the next-subvolume method [(Elf et al., 2003)](References#Elf03.md). The popular software packages Smartcell [(Ander et al., 2004)](References#Ander04.md) and MesoRD [(Hattne et al., 2005)](References#Hattne05.md) can handle the next-reaction and the next-subvolume methods. The fastest implementation of the next-subvolume method so far is the stochastic simulation compiler [(Lis et al., 2009)](References#Lis09.md).

All algorithms in the previous paragraph compute _exact_ realizations of stochastic reaction-diffusion systems. The disadvantage of these methods is their high computational cost, in particular, if a fine granularity is required and the number of subvolumes is high. Furthermore, they do not lend themselves well for data-parallel implementations. However, if one is willing to sacrifice accuracy for speed, deterministic-stochastic _hybrid_ algorithms provide an efficient alternative. In these methods, the diffusion part is separated from the reaction part and treated deterministically. The Gillespie multiparticle method (GMP) [(Rodriguez et al., 2006)](References#Rodriguez06.md) is one example of this class of algorithms.

![http://img208.imageshack.us/img208/1723/reactiondiffusion.png](http://img208.imageshack.us/img208/1723/reactiondiffusion.png)

GMP handles the diffusion part of the reaction-diffusion network trough a multiparticle lattice gas automaton [(Chopard et al., 1994)](References#Chopard94.md) and the reaction part via Gillespie's direct method [(Gillespie, 1977)](References#Gillespie77.md). The time step is chosen globally according to the diffusion constants of each species and the dimensions of each lattice cell. In each iteration, all reactions are performed independently in each subvolume. At the end, all particles are diffused according to the cellular automaton rule.

# Implementation on GPGPUs #
With the advent of general-purpose graphics processing units (GPGPUs) there is potential to provide super-computing resources to a broad audience. GPUs are ubiquitous and comparably cheap and designated units can be mounted in work station computers for better performance. GPGPUs provide a multitude of processing units (threads) which are executed in parallel. Moreover, efficient latency hiding and zero-overhead scheduling allow for massively-parallel data processing and can thus be efficiently used for scientific computing applications. GPU threads are very light-weight and work most efficiently when the same instruction is executed by all threads in a warp (single-instruction multiple-data).

We present a data-parallel implementation of GMP on GPGPUs (GPGMP). Since all cells are treated independently during each time step, our approach is straight-forward. Each thread works independently on exactly one subvolume. Global synchronization is only required at the end of the diffusion time step to exchange information about particles moving in between neighbouring cells. We refer the reader to [Vigelius et al. (in preparation)](References#Vigelius10.md) for implementation details.

<img src='http://img198.imageshack.us/img198/876/timing.png' border='0' />

GPGMP allows researchers to efficiently simulate spatially-inhomogeneous reaction-diffusion networks. Even if no access to designated GPGPU clusters is available, GPGMP can still provide considerable speed gains on built-in GPGPU cards on workstations (see the figure above for a comparison of several algorithms). Furthermore, we provide free access to the Monash Sun Grid GPGMP cluster through our easy-to-use web interface to GPGMP, [Inchman](InchmanTutorial.md).

# Getting started with GPGMP and Inchman #
We recommend to try out some of the features of GPGMP through [Inchman](InchmanTutorial.md). For more sophisticated models, however, it might be necessary to download the source code and compile your models from scratch. Detailed instructions and further information about GPGMP can be found [here](Usage.md).

If you find this software useful, we kindly ask you to reference [Vigelius et al. (in preparation)](References#Vigelius10.md) in your publication.

# Applications #
With time, we will add neat examples of GPGMP applications here. If you have used GPGMP for your research, we would love to hear from you!