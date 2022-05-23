---
title: 'SDE simulation with complex non-diagonal noise on DifferentialEquations.jl'
date: 2022-05-22
permalink: /posts/2022/05/blog-post-SDE-complex/
tags:
  - Programming
  - Julialang
  - Quantum Optics
  - Quantum Mechanics
---

The aim of this post is to keep a note on a pitfall that I felt in when implementing a certain type of stochastic differential equation (SDE) using [DifferentialEquations.jl](https://diffeq.sciml.ai/stable/tutorials/sde_example/), recapping the discussions that took place [here](https://github.com/SciML/DifferentialEquations.jl/issues/865){:target="_blank"}.

The type of SDE that we are interested in here is a multi-variable complex SDE with multiple real noise processes

$$\mathrm{d}u_i=\sum_{j=1}^mc_{ij}\,\mathrm{d}W_j$$

where $$c_{ij}$$ take complex values, and $\mathrm{d}W_j$ are real Wiener increments. The indices run through $$i\in \{1,\cdots,n\}$$ and $$j\in \{1,\cdots,m\}$$. (I have encountered an SDE of this form when writing a solver for a [stochastic Schrödinger equation](https://docs.qojulia.org/stochastic/schroedinger/) with multiple dissipation channels under homodyne unraveling. It is essential that the variables are complex-valued to handle a complex vector representing a quantum state, and the noises have to be real Wiener processes to account for real-valued homodyne records.) 

For simplicity, let us consider the case with $$n=m=2$$ with $c_{ij}=\mathrm{i}$ in the following. A na\"ive implementation that I did first is shown below.


        using DifferentialEquations

        function f(du,u,p,t)
            du .= 0.0im
        end

        function g(du,u,p,t)
            du[1,1] = 1.0im
            du[1,2] = 1.0im
            du[2,1] = 1.0im
            du[2,2] = 1.0im
        end

        u0 = [0.0im,0.0im]
        dt = 0.001
        prob = SDEProblem(f,g,u0,(0.0,1.0),noise_rate_prototype=zeros(2,2))
        sol = solve(prob,EM(),dt=dt)

Here, the aim is to use the <code>noise_rate_prototype</code> argument to handle more than one independent noise processes. A generic case of $$c_{ij}$$ can be dealt, e.g., with <code>du[i,j] = c_ij</code> and <code>noise_rate_prototype=zeros(n,m)</code>. 


While the code works perfectly for real $$c_{ij}$$, it actually gives us an "InexactError" when we use complex-valued $$c_{ij}$$'s. This is because the type of the stochastic contribution is set by the type of <code>noise_rate_prototype</code>, instead of the type of the initial state <code>u0</code>. In order to enable complex-valued stochastic increments, we instead need to do 


        prob = SDEProblem(f,g,u0,(0.0,1.0),noise_rate_prototype=zeros(ComplexF64,2,2))

Although the code above runs without an error this time, it turns out that it still does not achieve our original goal; After running the code, one will find that the <code>sol.u</code> contains finite real components, while our intended SDE should lead to a purely imaginary <code>sol.u</code>. This is because setting <code>noise_rate_prototype</code> to a complex matrix also renders the noise processes (i.e., $\mathrm{d}W_j$) complex Wiener process (which might be what one wants sometimes).

To implement an SDE with complex-valued stochastic terms driven by real Wiener processes, we need to manually set the <code>noise</code> to a real one, which we can do, e.g., with the code below.

        
        W = WienerProcess!(0.0,zeros(2),zeros(2))
        prob = SDEProblem(f,g,u0,(0.0,1.0),noise_rate_prototype=zeros(ComplexF64,2,2),noise=W)

Note that cases with generic $$m$$ can be handled by <code>W = WienerProcess!(0.0,zeros(m),zeros(m))</code>.