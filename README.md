## Sparse DMD in Python

The Dynamic Mode Decomposition is a tool for analysing spatially
distributed time-series, motivated by seeking recurring patterns in
2D velocity data from experiments in fluids.

The DMD finds the best fit to the data with a number of 'dynamic'
modes, each having a distinct frequency of oscillation.

A drawback of the standard DMD is that the number of modes is the
same as the number of fields in the decomposition axis of the data
and that there is no clear way to select which of these modes best
represent the data.

The sparse DMD (Jovanovic et al, 2014) aims to find a reduced number
of dynamic modes that best represent the data.

This is a Python version of the reference [matlab source][matlab_source].

[matlab_source]: http://www.ece.umn.edu/users/mihailo//software/dmdsp/download.html


### Usage

Assuming that `u` is some 3d array containing 2d velocity data
through time, we use a convenience method to create the matrix of
snapshots and compute the standard DMD:

```python
import sparse_dmd

# load data
u = load_some_data()

# create snapshots, using last array axis for decomposition
snapshots = sparse_dmd.to_snaps(u, decomp_axis=-1)

# create and compute the standard dmd
dmd = sparse_dmd.SparseDMD(snapshots)
```

Now we can compute the sparse dmd, given some parameterisation
range:

```python
# create range of sparsity parameterisation
gamma = np.logspace(-2, 6, 200)

# compute the sparse dmd using this gamma range
dmd.compute_dmdsp(gamma)
```

You may have to tweak the range of `gamma` manually, until it nicely
covers your data.

You can now access the results of the sparsity computation:

```python
# optimal mode amplitudes
optimal_amplitudes = dmd.sparse.xpol

# number of non-zero amplitudes
dmd.sparse.Nz
```

#### Plotting

The plotting routines from the matlab source have been copied over
and can easily be performed:

```python
import matplotlib.pyplot as plt

plotter = sparse_dmd.SparsePlots(dmd)

fig = plotter.performance_loss_gamma()
plt.show()

fig, ax = plt.subplots()
plotter.nonzero_gamma(ax)
```

#### Reconstruction

Given a sparse computation and a desired number of modes we can
attempt to reconstruct the original data.

Currently you have to perform the computation with given gamma and
look the array of the number of nonzero amplitudes, `dmd.sparse.Nz`,
for the index into `gamma` corresponding to the number of modes that
you want (e.g. `Ni=30` here).

```python
# compute the reconstruction for gamma[30]
dmd.compute_sparse_reconstruction(Ni=30, shape=u.shape, decomp_axis=-1)

reconstructed_data = dmd.reconstruction.rdata

reduced_set_of_modes = dmd.reconstruction.modes
reduced_set_of_ritz_values = dmd.reconstruction.freqs
reduced_set_of_amplitudes = dmd.reconstruction.amplitudes
```


### Performance

This is not as fast as the matlab source (~30% speed), despite my
best efforts at optimisation of the ADMM method which forms the
innermost loop.


### Contributing

Very welcome, especially for performance! Just create an issue /
open a pull request.
