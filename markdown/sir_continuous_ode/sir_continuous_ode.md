The classical ODE version of the SIR model is:

- Deterministic
- Continuous in time
- Continuous in state

$$
\frac{dS}{dt} = -\beta \frac{SI}{N} \
\frac{dI}{dt} = \beta \frac{SI}{N} - \gamma I \
\frac{dR}{dt} = \gamma I \
N = S+I+R
$$

````julia
using DifferentialEquations
using SimpleDiffEq
using Plots
using BenchmarkTools
````





The following function provides the derivatives of the model, which it changes in-place. State variables and parameters are unpacked from `u` and `p`; this incurs a slight performance hit, but makes the equations much easier to read.

````julia
function sir_ode!(du,u,p,t)
    (S,I,R) = u
    (β,γ) = p
    N = S+I+R
    @inbounds begin
        du[1] = -β*S*I/N
        du[2] = β*S*I/N - γ*I
        du[3] = γ*I
    end
    nothing
end;
````





We set the timespan for simulations, `tspan`, initial conditions, `u0`, and parameter values, `p` (which are unpacked above as `[β,γ]`).

````julia
tspan = (0.0,40.0)
u0 = [990.0,10.0,0.0]
p = [0.5,0.25];
````



````julia
prob_sir_ode = ODEProblem(sir_ode!,u0,tspan,p)
````


````
ODEProblem with uType Array{Float64,1} and tType Float64. In-place: true
timespan: (0.0, 40.0)
u0: [990.0, 10.0, 0.0]
````



````julia
sol_sir_ode = solve(prob_sir_ode);
````



````julia
plot(sol_sir_ode,vars=[(0,1),(0,2),(0,3)])
````


![](figures/sir_continuous_ode_6_1.png)

````julia
@benchmark solve(prob_sir_ode)
````


````
BenchmarkTools.Trial: 
  memory estimate:  31.23 KiB
  allocs estimate:  334
  --------------
  minimum time:     35.801 μs (0.00% GC)
  median time:      61.100 μs (0.00% GC)
  mean time:        74.487 μs (6.04% GC)
  maximum time:     15.740 ms (99.13% GC)
  --------------
  samples:          10000
  evals/sample:     1
````

