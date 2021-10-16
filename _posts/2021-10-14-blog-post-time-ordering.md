---
title: 'On Hermitian conjugation of time-ordered unitary'
date: 2021-10-14
permalink: /posts/2021/10/blog-post-time-ordering/
tags:
  - Quantum Mechanics
---

Taking Hermitian conjugation of an operator is an everyday task in quantum mechanics, while more care is needed when handling operators with some nontrivial structures. Here, this post addresses a pitfall that I recently fell in in an attempt to take a Hermitian conjugate of a unitary involving time-ordering. While the content of this post might be totally obvious for some, I have still managed to trick non-trivial number of people with years of experience in quantum mechanics with this problem.

------
A general rule of thumb when taking a Hermitian conjugate of an operator is to put daggers on every operators involved and replace $$i$$ with $$-i$$. Although this procedure nominally works well, a seemingly innocent example that I stumbled upon recently was a unitary involving time-ordering

$$ U(t)=\mathcal{T}\exp\left(-i\int^t_0\mathrm{d}t'H(t')\right), $$

which represents propagation under a time-dependent Hamiltonian $$H(t)$$ from time $$0$$ to $$t$$. Applying the procedures mentioned above, it seems that

$$ U^\dagger(t)\stackrel{?}{=}\mathcal{T}\exp\left(i\int^t_0\mathrm{d}t'H(t')\right) $$

holds true, since $$H^\dagger=H$$. 

It turns out, however, that it is not that simple, due to the presence of the time-ordering operator $$\mathcal{T}$$. To see the issue more clearly, we need to expand the expression of $$U(t)$$ in its full form as

$$U(t)=\sum_{n=0}^\infty(-i)^n\int^t_0\mathrm{d}t_1\int^{t_1}_0\mathrm{d}t_2\cdots \int^{t_{n-1}}_0\mathrm{d}t_n\,H(t_1)H(t_2)\cdots H(t_n).$$

Now, by taking the Hermitian conjugate, we get 

$$U^\dagger(t)=\sum_{n=0}^\infty(i)^n\int^t_0\mathrm{d}t_1\int^{t_1}_0\mathrm{d}t_2\cdots \int^{t_{n-1}}_0\mathrm{d}t_n\,H(t_n)H(t_{n-1})\cdots H(t_1),$$

where we have used $$(H(t_1)H(t_2)\cdots H(t_n))^\dagger=H(t_n)H(t_{n-1})\cdots H(t_1)$$. Notice that the fact that Hamiltonians at different times do not necessarily commute lead to an inversion of time ordering in the integral. Formally, we can denote the correct answer as

$$ U^\dagger(t)=\mathcal{A}\exp\left(i\int^t_0\mathrm{d}t'H(t')\right),$$

where $$\mathcal{A}$$ is an "anti-time-ordering" operator.