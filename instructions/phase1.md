# Phase 1

This phase of construction is to develop the testing infastructure prior to connecting the chip.

# Prerequisites
- a linux workstation
  - vivado 2022.2 (or newer)
  - miniconda or anaconda
```
export PROJECT_DIR=/asic/projects/C/CMS_PIX_28/testing/
mkdir -p $PROJECT_DIR/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O $PROJECT_DIR/miniconda.sh
bash $PROJECT_DIR/miniconda.sh -b -u -p $PROJECT_DIR/miniconda3
rm -rf $PROJECT_DIR/miniconda.sh
$PROJECT_DIR/miniconda3/bin/conda init bash
```
- Xilinx ZCU102 with petalinux installed onto SD card. ssh into FPGA setup. https://www.xilinx.com/products/boards-and-kits/ek-u1-zcu102-g.html
- Xilinx FMC XM105 Debug Card (mezzanine) https://www.xilinx.com/products/boards-and-kits/hw-fmc-xm105-g.html
- oscilloscope
- voltmeter 
- breadboard cables to connect pins on mezzanine
- cables to connect pins of mezzanine to oscilloscope

# Set up the DAQ 
1. clone the repo following instructions in readme https://github.com/badeaa3/CMSPIX28_DAQ
  - report to Anthony Badea (anthony.badea@cern.ch) any repository that you do not access to or are unable to clone

2. setup spacely virtual environment for spacely
```
cd CMSPIX28_DAQ/spacely/PySpacely/requirements
conda create --name spacelyvenv python=3.10
conda activate spacelyvenv
pip install -r requirements-py-libs-common.txt
pip install -r requirements-python.txt
conda activate spacelyvenv
```
- cd CMSPIX28_DAQ/spacely/PySpacely/
- edit Master_Config.py target to "CMSPIX28"
- comment out lines XX in src/Spacely_Caribou.py

3. generate bitstream and flash FPGA
```
cd CMSPIX28_DAQ/cms_pix_28_test_firmware/vivado
vivado &
```
- turn on FPGA without mezzanine connected
- verify LEDs on FPGA then connect the mezzanine
- use breadboard pins to connect XX and XX on mezzanine
- Open project and select the .xdc file
- Let vivado load
- On the left, click "Generate Bitstream" ...
- File -> Export Hardware -> Continue ...
- Program Device -> autoconnect ...

4. setup peary FPGA
```
scp CMSPIX28_DAQ/peary petalinux@X:XX/XX
scp CMSPIX28_DAQ/cms_pix_28_test_firmware/device/CMSPIX28 petalinux@X:XX/device
scp cmake petalinux@X
ssh petalinux@X
export PATH=PATH:cmake
cd peary
mkdir build
cd build
CMAKE ...
make -j4
sudo make install
cd ../
sudo ./bin/pearyd
```

5. Run spacely on linux machine
```
cd CMSPIX28_DAQ/spacely/PySpacely/
python Spacely.py
[click enter]
```
You should see peary recognize the device


# Tests

## Test 0 no scope
- run peary, run spacely, run test routine 3 

## Test 1 with scope
- connect the correct mezzanine pins to the scope, clock divide test, should see clock frequency change

## Test 2 with scope
- connect the correct mezzanine pins to the scope, clock divide test, should see delay between bxclk and bxclk_ana change on scope

## Test 3 with scope
- set scope to single event mode
- run test to produce pulses on the scope