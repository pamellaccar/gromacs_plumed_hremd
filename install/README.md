# Install PLUMED 
```
wget https://github.com/plumed/plumed2/archive/refs/tags/v2.5.5.tar.gz
tar -xzf v2.5.5.tar.gz
cd plumed2-2.5.5
./configure --prefix=usr/plumed2
make -j 4
make install
```

# Install GROMACS with PLUMED
```
wget ftp://ftp.gromacs.org/pub/gromacs/gromacs-2022.1.tar.gz
tar -xzf gromacs-2022.1.tar.gz
cd gromacs-2022.1
plumed-patch -p -e gromacs-2022.1
mkdir build
cd build
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=OFF -DGMX_MPI=ON -DGMX_GPU=OFF -DCMAKE_C_COMPILER=gcc -DGMX_FFT_LIBRARY=fftpack -DCMAKE_INSTALL_PREFIX=$work/gromacs-2019.4
make -j 4
make install
source $PATH/gromacs-2022.1/bin/GMXRC
```

# Install PACKMOL
```
wget http://leandro.iqm.unicamp.br/m3g/packmol/packmol.tar.gz
tar -xzf packmol.tar.gz
cd packmol
make
```
