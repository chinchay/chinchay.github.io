---
title: First post
author: Carlos Leon
date: 2021-03-26 23:07:07 -0500
categories: [Blogging]
tags: [firstpost]
pin: true
---

## About this blog
Here I will post notes about coding, machine learning, and other stuff that might be useful in a dayly work routine or research.

For example, `ASE` is an amazing python package to work with atom structures. To create the narrowest armchair graphene nanoribbon type:

```python
vacuum = 9.0
from ase.build import graphene_nanoribbon
atoms = graphene_nanoribbon(3/2, 1, type='armchair', saturated=True, C_H=1.1, C_C=1.4, vacuum=vacuum,  magnetic=True, initial_mag=1.12)
```

and to show the structure in Visual Code (you would need Jupyter extension installed on VScode):
```python
import nglview as nv
v = nv.show_ase(atoms)
v.background = 'black'
v
```


