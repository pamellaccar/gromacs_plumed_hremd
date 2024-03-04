<h1 align="center"> GROMACS Simulations  </h1> 

Download the structure from the Protein Data Bank (PDB): In this case we're using 2muj.pdb, a Plasmodium falciparum peptide. 

<h3> First step: Initial configuration and topology, adding Hydrogens using pdb2gmx. </h3>

```gmx_mpi pdb2gmx -f 2muj.pdb -o 2muj_H.pdb -ignh```

select CHARMM36m force field and tip3p water model.


<h3> Second step: Building the box with PACKMOL. </h3>

The peptide has 41 Angstroms size, so we're going to define 15 Angstroms for each side of the box: 41+15=56 Angstroms approximately.

```./solvate.tcl 2muj_H.pdb -shell 15. -charge -1 -density 1.0 -o solvated.pdb```

paste into packmol input file:

``` tolerance 2.0

filetype pdb

seed -1

add_box_sides 1.0

output solvated.pdb

```

and run:

```packmol < packmol_input.inp```

Put the solvent and ion number in the end of the topology file (check the names in each force field). E.g.:

```
#ifdef POSRES_WATER
; Position restraint for each water oxygen
[ position_restraints ]
;  i funct       fcx        fcy        fcz
   1    1       1000       1000       1000
#endif

; Include topology for ions
#include "./charmm36-jul2021.ff/ions.itp"

[ system ]
; Name
SERINE-REPEAT ANTIGEN PROTEIN

[ molecules ]
; Compound        #mols
Protein_chain_A     1
SOL                4903
SOD                14
CLA                13
```


<h3> Third step: Build mdp files for each force field and run minimization. </h3>

mdp files: https://manual.gromacs.org/documentation/current/user-guide/mdp-options.html

```gmx_mpi grompp -f mim.mdp -c solvated.pdb -r solvated.pdb -p topol.top -o minim.tpr -pp processed.top -maxwarn 1```

```gmx_mpi mdrun -v -deffnm minim```


<h3> Fourth step: Run equilibration and choose ensembles. </h3>

From this step on, system replicas are created. We will use five replicas as a test. The command below copies the files into the created directories:

```
touch plumed.dat #this command will create an empty plumed file needed for replica exchanges
mkdir -p 0 1 2 3 4 
echo {0..4} | xargs -n 1 cp nvt.mdp npt.mdp md.mdp plumed.dat
```
To indicate which atoms we will apply the sampling acceleration method to, we have to modify the file `processed.top` to a new file called `processed_.top`, where we will add `_` in front of the name of all the atoms that will be included in sampling acceleration. In this case, we will include all atoms of the chosen peptide; however, it is possible to select only the regions that will be accelerated.

It is necessary to add `_` in front of the name of each protein atom (in `vim`, use Control-V, select all columns referring to protein atoms in the position following the atom name, and use `shift-i _ `).

```
[ atoms ]
;   nr       type  resnr residue  atom   cgnr     charge       mass  typeB    chargeB      massB
; residue   1 TYR rtp TYR  q +1.0
     1        NH3_      1    TYR      N      1       -0.3     14.007
     2         HC_      1    TYR     H1      2       0.33      1.008
     3         HC_      1    TYR     H2      3       0.33      1.008
     4         HC_      1    TYR     H3      4       0.33      1.008
     5        CT1_      1    TYR     CA      5       0.21     12.011
     6        HB1_      1    TYR     HA      5        0.1      1.008
     7        CT2_      1    TYR     CB      6      -0.18     12.011
     8        HA2_      1    TYR    HB1      6       0.09      1.008
     9        HA2_      1    TYR    HB2      6       0.09      1.008
    10         CA_      1    TYR     CG      7          0     12.011
    11         CA_      1    TYR    CD1      8     -0.115     12.011
    12         HP_      1    TYR    HD1      8      0.115      1.008
    13         CA_      1    TYR    CE1      9     -0.115     12.011
    14         HP_      1    TYR    HE1      9      0.115      1.008
    15         CA_      1    TYR     CZ     10       0.11     12.011
    16        OH1_      1    TYR     OH     10      -0.54    15.9994
    17          H_      1    TYR     HH     10       0.43      1.008
    18         CA_      1    TYR    CD2     11     -0.115     12.011
```
 

**NVT equilibration**

```
for dir in 0 1 2 3 4; do
  cd $dir
  gmx_mpi grompp -f nvt.mdp -c ../minim.gro -p topol.top -o canonical.tpr -maxwarn 1
  cd ..
done
```

```
mpirun -np 10 gmx_mpi mdrun -s canonical.tpr -v -deffnm canonical -multidir 0 1 2 3 4
```

**NPT equilibration**

```
for dir in 0 1 2 3 4; do
  cd $dir
  gmx_mpi grompp -f npt.mdp -c canonical.gro -p topol.top -o isobaric.tpr -maxwarn 1
  cd ..
done
```

```
mpirun -np 10 gmx_mpi mdrun -plumed plumed.dat -s md.tpr -v -deffnm md -multidir 0 1 2 3 4 -replex 100 -hrex -dlb no
```

<h3> Fifth step: MD production </h3> 

```
for dir in 0 1 2 3 4; do
  cd $dir
  gmx_mpi grompp -f md.mdp -p topol.top -c isobaric.gro -o md.tpr -maxwarn 1
  cd ..
done
```

```
mpirun -np 10 gmx_mpi mdrun -plumed plumed.dat -s md.tpr -v -deffnm md -multidir 0 1 2 3 4 -replex 100 -hrex -dlb no
```

Analyze results from replica 0:

```
cd 0
echo 1 0 |gmx_mpi trjconv -s md.tpr -f md.gro -o processed.gro -ur compact -pbc mol -center
echo 1 0 |gmx_mpi trjconv -s md.tpr -f md.xtc -o processed.xtc -ur compact -pbc mol -center
```

GROMACS reminds you: "Shit happens!" (Pulp Fiction)



