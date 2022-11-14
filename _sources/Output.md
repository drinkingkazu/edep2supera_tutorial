---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.10.3
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Understanding the Output
Let's take a look at the outpupt file `larcv.root` which we created in the last tutorial.

In particular, we go over the following topics.
* The contents of the data file
* Accessing `larcv` data products via `TChain`
* Visualization of 3D energy deposition
* Particle information
* Clusters of pixels

## Before Starting

All `larcv` data is stored in the form of multiple `ROOT::TTree` instances.
These TTrees have their entries aligned by a unique _event_ that is either simulated or recorded from a real detector.
In this tutorial, we access multiple TTrees for a particular event and compare the data contents from each TTree.
To avoid a confusion down the road, let's define both the filename and a particular entry to analyze here.

```{code-cell}
FILENAME = 'larcv.root'
ENTRY    = 0
```
## The Contents

Supera generates five data products in the output. 
* All voxelized energy depositions in MeV ... `larcv::EventSparseTensor3D` type, name `pcluster`
* All semantic types (discrete pixel categories) ... `larcv::EventSparseTensor3D` type, name `pcluster_semantic`
* Particle-wise particle information ... `larcv::EventParticle` type, name `pcluster`
* Particle-wise voxelized energy depositions in MeV ... `larcv::EventClusterVoxel3D` type, name `pcluster`
* Particle-wise voxelized dE/dX in MeV/cm ... `larcv::EventClusterVoxel3D` type, name `pcluster_dedx`

Let's look at the list of data products stored in the file.

```{code-cell}
from ROOT import TFile
f = TFile.Open(FILENAME,'READ')
f.ls()
```

Each TTree holds one `larcv` data product with an associated unique name.

This name is used as a key to retrieve a particular data product instance from a file.
Here is how to construct such a key:
```
$TYPE_$NAME_tree
```
in which you should replace `$TYPE` and `$NAME` with the right text.

`$TYPE` depends on the type of data product.
* "sparse3d" for `larcv::EventSparseTensor3D` type
* "cluster3d" for `larcv::EventClusterVoxel3D` type
* "particle" for `larcv::EventParticle` type

`$NAME` is a unique identifier for the specified type.

For instance, "all voxelized energy depositions in MeV" can be accessed with:
```
$TYPE = sparse3d
$NAME = pcluster
```
The corresponding TTree name is `sparse3d_pcluster_tree`.

## Access via `TChain`
Next, we take a brief look at one of the output data products.
These output data products can be accessed in Python using `ROOT::TChain`.

Let's try `sparse3d_pcluster_tree`:


```{code-cell}
from ROOT import TChain

energy_tree=TChain("sparse3d_pcluster_tree")
energy_tree.AddFile(FILENAME)

print(energy_tree.GetEntries(),'events found')
```

Events are discrete sets of a (simulated) detector response and have no correlations. This file holds 10 events because the input to Supera from the last tutorial held 10 events.
`sparse3d_pcluster_tree` is a recorded 3D image of all energy depositions that happened in each event.

Next, let's take a look at a specific event. To access a specific entry, we take 2 steps:
1. tell `TChain` which entry to access, and
2. access the corresponding data product.

For the latter, the data product is kept as a `TChain` attribute with the name:
```
$TYPE_$NAME_branch
```
which is a similar string we used last time except for `tree` replaced with `branch`.


```{code-cell}
# Access the first (0-th) entry in this file
energy_tree.GetEntry(ENTRY)

# Access the data product instance that is kept as TChain's attribute
energy_tensor = energy_tree.sparse3d_pcluster_branch

print(energy_tensor)
```

Now you can play with this `larcv::EventSparseTensor3D` however you want!

For instance, check out the number of 3D voxels stored:

```{code-cell}
print('Example entry has',energy_tensor.size(),'3D voxels')
```

## Visualizing a 3D Image of Energy Deposition
The `energy_tensor` extracted from `sparse3d_pcluster_tree` holds an array of voxels where energy is deposited. 

There is one data product per event, and it can be visualized as a 3D image.


But we don't remember all about this data type `larcv::EventSparseTensor3D`, and don't want to read the documentation...

Luckily, `larcv` provides a utility function called `fill_3d_pcloud` to copy the contents of `larcv::EventSparseTensor3D` into a numpy array.

A numpy array must be initialized with the right length = number of points, which you already know how to query :)

```{code-cell}
from larcv import larcv
import numpy as np

# Create a numpy array of the correct length
points=np.zeros(shape=(energy_tensor.size(),4),dtype=np.float32)
# Call larcv function to fill the numpy array
larcv.fill_3d_pcloud(energy_tensor,points)
```

Sweet! FYI: we made the second dimension size `4` to store `(x,y,z,value)` of each pixel, where the `value` is energy deposition in MeV for this particular data instance. If we specified it to `3`, only `(x,y,z)` will be stored. If we specified to `1`, only `(value)` is stored. These are features of `fill_3d_pcloud` function.

Let's visualize the stored energy depositions as a 3D scattered plot with `plotly`.

```{code-cell}
import plotly.graph_objs as go

# Create a scatter plot
trace = go.Scatter3d(x=points[:,0], y=points[:,1], z=points[:,2],
                     mode='markers',
                     marker=dict(size=1, color=points[:,3], opacity=0.3),
                    )
fig = go.Figure(data=trace)
fig.show()
```

## Semantic Label

Let's try another `EventSparseTensor3D` type data, `sparse3d_pcluster_semantic_tree`. 

This one contains a set of voxels that have the same locations as those stored in `sparse3d_pcluster` which we just played with. However, voxel values are different: each voxel stores `larcv::SemanticType_t` enumerator values (i.e. integers):
* `kShapeShower    = 0` ... shower-like particles
* `kShapeTrack     = 1` ... track-like particles
* `kShapeMichel    = 2` ... michel electrons
* `kShapeDelta     = 3` ... delta rays
* `kShapeLEScatter = 4` ... scattered small energy depositions
* `kShapeGhost     = 5` ... mis-reconstructed 3D points (only relevant for wire LArTPC)
* `kShapeUnknown   = 6` ... uncategorized type (it should not be present)
Identifying these types at individual voxel-level is one of key tasks in data reconstruction.

Let's play one of them! As you can see in the previous section, `sparse3d_pcluster` includes all energy depositions and the image is dominated by lots of fragmented energy depositions that are typically a single point. Removing those make it a lot easier to interpret the particle trajectories in the same image.

Below, we access `sparse3d_pcluster_semantic` and retrieve only the voxel values to generate a mask for voxels of the semantic type `kShapeLEScatter`.

```{code-cell}
semantic_tree=TChain("sparse3d_pcluster_semantic_tree")
semantic_tree.AddFile(FILENAME)

semantic_tree.GetEntry(ENTRY)
semantic_tensor = semantic_tree.sparse3d_pcluster_semantic_branch

print('Example entry has',semantic_tensor.size(),'3D points')

# Fill the semantic values
semantic_value=np.zeros(shape=(semantic_tensor.size(),1),dtype=np.float32)
larcv.fill_3d_pcloud(semantic_tensor,semantic_value)

# Reshape and generate a semantic mask
semantic_value = semantic_value.reshape(-1).astype(np.int32)
mask = np.where(~(semantic_value==larcv.kShapeLEScatter))

# Create a scatter plot
trace = go.Scatter3d(x=points[mask][:,0], y=points[mask][:,1], z=points[mask][:,2],
                     mode='markers',
                     marker=dict(size=1, color=points[mask][:,3], opacity=0.3),
                    )
fig = go.Figure(data=trace)
fig.show()
```

## Particle Information

In a similar fashion, we can also access particle-level information.

Let's retrieve `particle_pcluster_tree`.

```{code-cell}
particle_tree = TChain("particle_pcluster_tree")
particle_tree.AddFile(FILENAME)
particle_tree.GetEntry(ENTRY)

particles = particle_tree.particle_pcluster_branch

print(particles.size(),'particles in the entry',ENTRY)
```

We can access individual particle information as a `std::vector<larcv::Particle>` type by calling `larcv::EventParticle::as_vector` function. 

```{code-cell}
p = particles.as_vector()[0]

print('Found a particle with Track ID',p.track_id(),'PDG code',p.pdg_code(),'initial energy',p.energy_init(),'MeV')
```

See `larcv::Particle` definition for all list of attributes for this data product. Just FYI, if you want a brief summary of a particle, you can call `larcv::Particle::dump` function.

```{code-cell}
print(p.dump())
```

## Clusters of Pixels
Finally, let's try retrieving a set of voxels for individual particles. This is exactly what is stored in `cluster3d_pcluster_tree`. 

```{code-cell}
import plotly
colors = plotly.colors.qualitative.Light24

cluster_tree=TChain("cluster3d_pcluster_tree")
cluster_tree.AddFile(FILENAME)

cluster_tree.GetEntry(ENTRY)
larcv_data = cluster_tree.cluster3d_pcluster_branch

print(larcv_data.size(),'clusters found!')
```

`cluster3d_pcluster` is in the type `larcv::EventClusterVoxel3D`. This data product type stores an array of array of voxels. For this particular data product, there is one array of voxels stored per particle. So the length of the outer array, which we just printed out above, should match with the number of particles we printed out earlier.

In order to correlate an array of voxels from `cluster3d_pcluster` with a particle information from `particle_pcluster`, simply use the same index. These data products are ordered and aligned by the index.

Let's now visualize individual particle instance with a discrete color.

```{code-cell}
traces=[]

# Loop over an array of "array of voxels (for one particle instance)"
for index,cluster in enumerate(larcv_data.as_vector()):
    
    # Create a numpy array and fill with (x,y,z) coordinate
    points = np.zeros(shape=(cluster.size(),3),dtype=np.float32)
    larcv.fill_3d_pcloud(cluster, larcv_data.meta(), points)

    # Let's only visualize those particles with more than 10 voxels.
    if len(points) < 10: continue

    # Assign a discrete color from a plotly color pallete
    color = colors[index % (len(colors))]
    
    # Name a particle with track ID and PDG code
    name = 'TID %d PDG %d' % (particles.as_vector()[index].track_id(),
                              particles.as_vector()[index].pdg_code()
                             )
    
    traces.append(go.Scatter3d(x=points[:,0], y=points[:,1], z=points[:,2],
                               mode='markers',
                               name=name,
                               marker=dict(size=1, color=color, opacity=0.3),
                              )
                 )
fig = go.Figure(data=traces)
fig.show()
```



