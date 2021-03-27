---
title: Nested sampling calculator
author: Carlos Leon
date: 2021-03-27
categories: [Blogging, nested sampling, molecular dynamics]
tags: [tricks]
<!--pin: true-->
---

## Attempts to use NS calculator

Nested sampling package brings its own calculator (modified from LAMMPS calculator). The way I was using is ilustrated in the following code:

```python
from ase import Atom, Atoms
from lammpslib import LAMMPSlib # <<-- NS calculator
import os

cmds = ["pair_style    mlip  mlip.ini",\
        "pair_coeff    * * "       ]

positions = [ (0.0, 0.0, 0.0), (0.0, 0.0, 5.0) ]
atoms = Atoms( 'AlW', positions=positions , cell=(a, a, a), pbc=True )
struct.get_positions()

absoletPath2file = os.getcwd() + "/log.txt"
mylammps = LAMMPSlib(lmpcmds=cmds, atom_types={13:1, 74:2}, keep_alive=True, log_file=absoletPath2file)

atoms.set_calculator(mylammps)
atoms.get_potential_energy(atoms)

# atoms.set_calculator(mylammps)
atoms.get_forces(atoms)

# atoms.set_calculator(mylammps)
atoms._calc.get_forces(atoms)
```
where `13` and `74` are the atomic numbers of the elements defined in `atoms`. I think I should had used `atoms.get_potential_energy()` instead of `atoms.get_potential_energy(atoms)`. It is important the keyword `keep_alive=True`, otherwise the calculator would need to be attached repeatedly as shown in the comments above.

The `atoms_types` keyword is not needed in the LAMMSP calculator (LAMMPS determines it automatically). NS code asks for this keyword because it uses its own system to call the participant atoms in the calculation, i.e, `1`, `2`, ...

