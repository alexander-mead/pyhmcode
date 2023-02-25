## HMcode

To whom it may concern,

I coded this `Python` version of `HMcode-2020` (confusingly [Mead et al. 2021](https://arxiv.org/abs/2009.01858)) up quite quickly before leaving academia. It is written in pure `Python` and doesn't use any of the original Fortran code whatsoever. There is something amazing/dispiriting about coding something up in 3 days that previously took 5 years. A tragic last hoorah! At least I switched to `Python` eventually...

You might also be interested in [`pyhalomodel`](https://pypi.org/project/pyhalomodel/), upon which this code depends, which implements a vanilla halo-model calculation for any desired large-scale-structure tracer, with the user providing only the halo profiles. Alternataively, and very confusingly, you might be intereted in this [`pyhmcode`](https://pypi.org/project/pyhmcode/), which provides a wrapper around the original `Fortran` `HMcode` implementation.

To install, clone the repository, `cd` into the directory, and then
```
poetry install
```

I tested it against the `CAMB-HMcode` version for 100 random sets of cosmological parameters ($k < 10 h\mathrm{Mpc}^{-1}$; $z < 3$). The level of agreement between the two codes is as follows:
- LCDM: Mean error: 0.10%; Std error: 0.03%; Worst error; 0.21%
- k-LCDM: Mean error: 0.11%; Std error: 0.03%; Worst error; 0.23%
- w-CDM: Mean error: 0.10%; Std error: 0.03%; Worst error; 0.20%
- w(a)-CDM: Mean error: 0.13%; Std error: 0.06%; Worst error; 0.48%
- nu-LCDM: Mean error: 0.47%; Std error: 0.44%; Worst error; 2.01% (larger errors strongly correlated with neutrino mass)
- nu-k-w(a)-CDM: Mean error: 0.42%; Std error: 0.43%; Worst error; 2.02% (larger errors strongly correlated with neutrino mass)

These tests can be reproduced using the `tests/test.py` script.

Note that the quoted accuracy of `HMcode` relative to simulations is RMS ~2.5%. Note also that the accuracy is anti-correlated with neutrino masses (cf. Fig. 2 of [Mead et al. 2021](https://arxiv.org/abs/2009.01858)). The larger discrepancies for massive neutrinos (2% for ~1eV) may seem worrisome, but here are some reasons why I am not that worried:
- Here neutrinos are treated as completely cold matter when calculating the linear growth factors whereas in `CAMB-HMcode` the transition from behaving like radiation to behaving like matter is accounted for in the linear growth.
- Here the cold matter power spectrum is taken directly from `CAMB` whereas in `CAMB-HMcode` the *cold* matter power spectrum is calculated approximately using [Eisenstein & Hu (1999)](https://arxiv.org/abs/astro-ph/9710252).

Using the actual cold matter spectrum is definitely a (small) improvement. While ignoring the actual energy-density scaling of massive neutrinos might seem to be a (small) problem, but keep in mind the comments below regarding the linear growth factor.

I think any residual differences must stem from:
- The BAO de-wiggling process
- The $\sigma_\mathrm{v}$ numerical integration
- The $\sigma(R)$ numerical integration (using `CAMB` here; done internally in `CAMB-HMcode`)
- The linear growth ODE solutions
- Root finding for the halo-collapse redshift and for $R_\mathrm{nl}$

But I didn't have time to investigate these differences more thoroughly. Note that there are accuracy parameters in `CAMB-HMcode` fixed at the $10^{-4}$ level, so you would never expect better than 0.01% agreement. Although given that `HMcode` is only accurate at the ~2.5% level compared to simulations the level of agreement between the codes seems okay, with the caveats about very massive neutrinos.

While writing this code I had a few ideas for future improvements:
- Add the `HMcode-2020` baryon-feedback model; this would not be too hard for the enthusiastic student/postdoc.
- The predictions are a bit sensitive to the smoothing $\sigma$ used for the dewiggling. This should probably be a fitted parameter.
- It's annoying having to calculate linear growth functions (all, LCDM), especially since the linear growth doesn't really exist. One should probably should use the $P(k)$ amplitude evolution over time at some cleverly chosen scale instead, or instead the evolution of $\sigma(R)$ over time at some pertinent $R$ value. Note that the growth factors are *only* used to calculate the [Dolag et al. (2004)](https://arxiv.org/abs/astro-ph/0309771) correction and [Mead (2017)](https://arxiv.org/abs/1606.05345) $\delta_\mathrm{c}$, $\Delta_\mathrm{v}$.
- I never liked the halo bloating parameter, it's hard to understand the effect of modifying halo profiles in Fourier space. Someone should get rid of this (maybe modify the mass function instead?).
- Redshift 'infinity' for the Dolag correction is actually $z_\infty = 10$. `HMcode` predictions *are* sensitive to this (particularly w(a)CDM). Either the redshift 'infinity' should be fitted or the halo-concentration model for beyond-LCDM should be improved somehow.
- The massive neutrino correction for the [Mead (2017)](https://arxiv.org/abs/1606.05345) $\delta_\mathrm{c}$, $\Delta_\mathrm{v}$ formula (appendix A of [Mead et al. 2021](https://arxiv.org/abs/2009.01858)) is crude and should be improved somehow. I guess using the intuition that hot neutrinos are ~smoothly distributed on halo scales. Currently neutrinos are treated as cold matter in the linear/accumulated growth calculation (used by [Mead 2017](https://arxiv.org/abs/1606.05345)), which seems a bit wrong.
- I haven't checked how fast this code is, but there are a couple of TODO in the code that might improve speed if necessary.
- The choices regarding how to account for massive neutrinos could usefully be revisited. This whole subject is a bit confusing and the code doesn't help to alleviate the confusion. Choices like what to use for: $\delta_\mathrm{c}$; $\Delta_\mathrm{v}$; $\sigma(R)$; $R_\mathrm{nl}$; $n_\mathrm{eff}$; $c(M)$.
- Refit model (including $\sigma$ for BAO smoothing and $z_\infty$ for [Dolag et al. 2004](https://arxiv.org/abs/astro-ph/0309771)) to new emulator(s) (e.g., [Mira Titan IV](https://arxiv.org/abs/2207.12345)).
- Don't be under any illusions that the `HMcode` parameters, or the forms of their dependence on the underlying power spectrum, are special in any particular way. A lot of experimentation went into finding these, but it was by no means exhaustive. Obviously these parameters should only depend on the underlying spectrum though (rather than being random functions of $z$ or whatever).

Have fun,

Alexander Mead (2023/02/28)
