# Molecular Dynamics with enhanced sampling tutorial using GROMACS and PLUMED

In my master's thesis I used Hamiltonian replica exchange molecular dynamics (HREMD) to understand the interaction mechanism of two proteins: an epitope of the *Plasmodium falciparum* AMA1 protein and a monoclonal antibody. In this case, AMA1 has flexible regions, and the enhanced sampling avoids the molecule being trapped in energy barriers with non-representative conformations. Replica exchange was applied to the entire epitope and only to the antigenic site regions of the antibody.

Check the full thesis here: [Estudo de Dinâmica Molecular do mecanismo de interação antígeno-anticorpo da proteína AMA1 do Plasmodium falciparum](https://www.repositorio.unicamp.br/acervo/detalhe/1375042?guid=1709124927980&returnUrl=%2fresultado%2flistar%3fguid%3d1709124927980%26quantidadePaginas%3d1%26codigoRegistro%3d1375042%231375042&i=1) 


In the next folders, you will find a step-by-step tutorial to run Molecular Dynamics Simulations with enhanced sampling of a pepitide using PLUMED implemented in GROMACS to perform replica exchanges and solvating the molecule with PACKMOL.

GROMACS: https://www.gromacs.org/

PACKMOL: http://leandro.iqm.unicamp.br/m3g/packmol/home.shtml

m3ggroup: https://github.com/m3g

See this course about this topic: *Simulação do enovelamento de proteínas e efeitos de solvente* https://github.com/m3g/XEMMSB2021/tree/main
