# performance
Benchmarking Results as of Jan 2023

- Hermit=True is ~50% quicker than Hermit=False in FMChain example (it's not easy to test with the BiFeO3 as Hermit=True doesn't work in the supercell).
- Using .mex will dramatically speed up the code (in the FMchain example it sped up the code by a factor of ~4 for all optmem and using .mex on BiFeO3 incommensurate calc. (no supercell) sped up the calculation by factor ~2)
- For many q-points (e.g. the FMChain) there is a bottle neck relating to matrix multiplication done in a loop over q-points (the matrices being multiplied depend on hemit = true/false but in both instances is responsible for >70% of the execution time)

Hermit: https://github.com/SpinW/spinw/blob/2341834ca326acaf7ff15896ef98d8594015dca4/swfiles/%40spinw/spinwave.m#L805-L807

Non-Hermit: https://github.com/SpinW/spinw/blob/2341834ca326acaf7ff15896ef98d8594015dca4/swfiles/%40spinw/spinwave.m#L868-L874

- For many magnetic atoms the bottleneck is diagonalising a 2nmag x 2*nmag matrix (where nmag is the number of mag atoms in the supercell if present- not the structural unit cell) for each q-point. This diagonalisation takes ~ 60% of the execution time in the BiFeO3 examples when using the mex version (calls `eig_omp`).This bottleneck doesn't necessarily scale linearly with the number of q-points because there is a .mex file that does a loop over a chunk of q-points, the size of the chunk is determined by the optmem argument - but in principle this could also be a bottlneck in systems with more modest nmag but a large number of q-points (e.g. in the FMChain example with mex not enabled this takes ~80% of the execution time).

https://github.com/SpinW/spinw/blob/2341834ca326acaf7ff15896ef98d8594015dca4/swfiles/%40spinw/spinwave.m#L864



- There are a handful of other matrix multiplication operations that each account for (2-5% of the execution time) that could be improved by using mex function sw_mtimesx if mex is enabled. For example

https://github.com/SpinW/spinw/blob/2341834ca326acaf7ff15896ef98d8594015dca4/swfiles/%40spinw/spinwave.m#L862
