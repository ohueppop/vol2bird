```bash
# instructions for Ubuntu-based systems

# cd to project directory

# define the root diretory of the project
RADAR_ROOT_DIR=${PWD}

# prepare a directory structure:
mkdir ${RADAR_ROOT_DIR}/opt
mkdir ${RADAR_ROOT_DIR}/src

# we'll use Python virtualenv to manage the python version, and this project's dependencies
sudo apt-get install python-virtualenv

# install Tcl and Tk for user interface
sudo apt-get install tcl-dev
sudo apt-get install tk-dev

# install HDF5, Hierarchichal Data Format library
sudo apt-get install libhdf5-dev

# install dependencies for pycurl
sudo apt-get install libcurl4-gnutls-dev
sudo apt-get install libgnutls-dev

# install library that can handle different projections
sudo apt-get install libproj-dev

# install package for xmlparsing of projection definitions
sudo apt-get install libexpat1-dev

# install library for parsing options
sudo apt-get install libconfuse-dev

# update the system database to be able to locate the files later:
sudo updatedb

# SKIPPED create a python2 virtualenv
virtualenv -p /usr/bin/python2.7 ${RADAR_ROOT_DIR}/.venv

# SKIPPED activate the python virtual environment if necessary (don't forget the leading dot):
. ${RADAR_ROOT_DIR}/.venv/bin/activate

# install libicu for unicode support
sudo apt-get install libicu55 libicu-dev

# update pip to the latest version
sudo pip install --upgrade pip

# install Numpy
sudo pip install numpy

# SKIPPED install Python Imaging Library (Pillow) into the virtual environment
sudo pip install Pillow

# SKIPPED install python inotify for watching changes on files
pip install pyinotify

# change to the source directory...
cd ${RADAR_ROOT_DIR}/src

# you'll need a copy of hlhdf:
git clone git://git.baltrad.eu/hlhdf.git

# change directory into the clone of hlhdf
cd hlhdf

# ./configure hlhdf
# we need to point to the location of the hdf5 headers and binaries
# find out where the headers live on your system with
locate hdf5.h
# for Ubuntu 14.04, the location is /usr/include/
# for Ubuntu 16.04, the location is /usr/include/hdf5/serial (might be different on your system)
#
# and the same for the binaries:
locate libhdf5.a  (static)
locate libhdf5.so (dynamic)
# for Ubuntu 14.04, the location is /usr/lib/x86_64-linux-gnu/
# for Ubuntu 16.04, the location is /usr/lib/x86_64-linux-gnu/hdf5/serial/ (might be different on your system)

./configure --prefix=${RADAR_ROOT_DIR}/opt/hlhdf --with-hdf5=/usr/include/,/usr/lib/x86_64-linux-gnu

# compile hlhdf5 in the local directory
make

# (optional) compile hlhdf5 tests and run them
make test

# copy the binaries to where we want them (${RADAR_ROOT_DIR}/opt/hlhdf):
make install

# SKIPPED create a script that sets up the environment so that Python can find _pyhlmodule.so and libhlhdf.so
echo ". ${RADAR_ROOT_DIR}/.venv/bin/activate" >${RADAR_ROOT_DIR}/setup-env
echo "export PYTHONPATH=\${PYTHONPATH}:${RADAR_ROOT_DIR}/opt/hlhdf/lib" >>${RADAR_ROOT_DIR}/setup-env
echo "export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:${RADAR_ROOT_DIR}/opt/hlhdf/lib" >>${RADAR_ROOT_DIR}/setup-env

# SKIPPED activate the environment (do this first every time you use vol2bird)
cd ${RADAR_ROOT_DIR}
. setup-env

# SKIPPED cd to the source directory
cd ${RADAR_ROOT_DIR}/src

# get a copy of rave:
git clone git://git.baltrad.eu/rave.git

# cd into rave source directory
cd rave

# rave needs keyczar to manage keys
sudo pip install python-keyczar

# install package for downloading
sudo pip install pycurl

# sanity checks
python -c 'import numpy'
python -c 'import _pyhl'

# SKIPPED set some environment variables
export RAVEROOT=${RADAR_ROOT_DIR}/opt/rave
export NUMPYDIR=${RADAR_ROOT_DIR}/.venv/lib/python2.7/site-packages/numpy/core/include/numpy
export HLDIR=${RADAR_ROOT_DIR}/opt/hlhdf
export PROJ4ROOT=/usr

# SKIPPED for building extensions, python needs to know about config; virtualenv does not set that
# up, so we'll symlink to the system python ourselves
# (64-bit)
ln -s /usr/lib/python2.7/config-x86_64-linux-gnu ${RADAR_ROOT_DIR}/.venv/lib/python2.7/config
# (32-bit)
# ln -s /usr/lib/python2.7/config ${RADAR_ROOT_DIR}/.venv/lib/python2.7/config

# now we're ready to configure the install
./configure --prefix=${RADAR_ROOT_DIR}/opt/rave --with-expat

#
make

#
make test

#
make install

# SKIPPED
echo "export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:${RADAR_ROOT_DIR}/opt/rave/lib" >>${RADAR_ROOT_DIR}/setup-env
echo "export PATH=\${PATH}:${RADAR_ROOT_DIR}/opt/rave/bin" >>${RADAR_ROOT_DIR}/setup-env

# SKIPPED
echo "${RADAR_ROOT_DIR}/opt/rave/Lib" > ${RADAR_ROOT_DIR}/.venv/lib/python2.7/site-packages/rave.pth

# SKIPPED
# cd back to the project root slash source
cd ${RADAR_ROOT_DIR}/src

# get a copy of vol2bird
git clone https://github.com/adokter/vol2bird.git

# change directory into it
cd vol2bird

# configure vol2bird
./configure 

#
make

# NOT WORKING
# optional: run tests
# note that we need to set some directories explicitly here, as there is no good
# way to autodetect these, and the vol2bird tests default to assuming you have RAVE
# installed as part of a full standard Baltrad installation
make test PREFIX=${RADAR_ROOT_DIR}/opt RAVEPYTHON=${RADAR_ROOT_DIR}/.venv2/bin/python

#
make install

# add vol2bird stuff to PATH
echo "export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:${RADAR_ROOT_DIR}/opt/vol2bird/lib" >>${RADAR_ROOT_DIR}/setup-env
echo "export PATH=\${PATH}:${RADAR_ROOT_DIR}/opt/vol2bird/bin" >>${RADAR_ROOT_DIR}/setup-env

# SKIPPED
# activate the environment (do this first every time you use vol2bird)
cd ${RADAR_ROOT_DIR}
. setup-env
