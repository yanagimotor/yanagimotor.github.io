---
title: 'Handling LAPACK exceptions in Julia SVD'
date: 2021-06-08
permalink: /posts/2021/06/blog-post-lapack/
tags:
  - Programming
  - Julialang
---

I have run numerical simulations that involve a large number of singular value decompositions (SVDs) using Julia, where I noticed that SVD algorithm fails on a few specific matrices out of several millions. While the issue can be economically circumvented by switching SVD algorithms whenever they fail, exact cause of the error has not been clear yet.

LAPACK exceptions
------
Implementing an SVD in [Julia](https://julialang.org) is as easy as

        F=svd(M)

which, most of the times, returns a proper SVD of M without any problem. However, as we run large number of SVDs on millions of matrices, we could sometimes stumble upon a nasty matrix that causes a LAPACKexception that looks like

        LAPACKException(1)

        Stacktrace:
        [1] chklapackerror(ret::Int32)
          @ LinearAlgebra.LAPACK /opt/julia-1.6.0/share/julia/stdlib/v1.6/LinearAlgebra/src/lapack.jl:38
        [2] gesdd!(job::Char, A::Matrix{ComplexF64})
          @ LinearAlgebra.LAPACK /opt/julia-1.6.0/share/julia/stdlib/v1.6/LinearAlgebra/src/lapack.jl:1670
        [3] _svd!
          @ /opt/julia-1.6.0/share/julia/stdlib/v1.6/LinearAlgebra/src/svd.jl:104 [inlined]
        [4] svd!(A::Matrix{ComplexF64}; full::Bool, alg::LinearAlgebra.DivideAndConquer)
          @ LinearAlgebra /opt/julia-1.6.0/share/julia/stdlib/v1.6/LinearAlgebra/src/svd.jl:97
        [5] #svd#85
          @ /opt/julia-1.6.0/share/julia/stdlib/v1.6/LinearAlgebra/src/svd.jl:157 [inlined]
        [6] svd(A::Matrix{ComplexF64})
          @ LinearAlgebra /opt/julia-1.6.0/share/julia/stdlib/v1.6/LinearAlgebra/src/svd.jl:157
        [7] top-level scope
          @ In[94]:1
        [8] eval
          @ ./boot.jl:360 [inlined]
        [9] include_string(mapexpr::typeof(REPL.softscope), mod::Module, code::String, filename::String)
          @ Base ./loading.jl:1094

I attach an example of such a matrix M <a href="/files/lapack_mat.jld" download>here</a>, whose elements are approximately zero except for M[1,1]~1. The exception seems to be caused by some specific structures of the matrix, but its occurrence depends on a very fine balance among matrix elements: For instance, we can eliminate the error by varying a single entry of the matrix by a tiny amount (say, by 1e-15). 

While the seemingly random occurrence of the error is hard to predict, circumventing the issue itself turns out to be possible without much modifications to the code: We can simply switch the SVD algorithm whenever one fails on a matrix (as suggested [here](https://discourse.julialang.org/t/lapackexception-1-while-svd-but-not-svdvals/23787)), which, in a concrete implementation, looks like

        F=SVD{ComplexF64, Float64, Matrix{ComplexF64}}
        try
            F=svd(mat,alg=LinearAlgebra.DivideAndConquer())
        catch e
            F=svd(mat,alg=LinearAlgebra.QRIteration())
        end;

Aside from the time consumption, the discrepancy between QRIteration() and DivideAndConquer() is negligible for practical uses.