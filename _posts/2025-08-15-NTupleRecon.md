---
title: Repository of NTuple Reconstruction
math: true
permalink: /NTuple Reconstruction/
---
## Overview
[**NTupleRecon**](https://gitlab.cern.ch/atlas-ttbar-10-angle/ntuplerecon) provides an automated workflow for reconstructing and merging $$t\bar{t}$$ NTuple slices produced by the CERN grid. The $$t\bar{t}$$ decaying process was fully simulated (including the detector effects) by the [Top MC production group](https://atlas-topq.docs.cern.ch/), resulting in the *DAOD* files distributed on the CERN grid. At this stage, the only available information for each event is its energy, momentum, and charge. Then DAOD signals are processed by the [TopCPToolkit](https://topcptoolkit.docs.cern.ch/latest/), where researchers define selection criteria — such as minimum transverse momentum and b-tagging algorithms — to pre-select candidate events relevant to the physics process of interest, such as $$t\bar{t}$$ decay. The output of this stage is referred to as *NTuple* files.

Although NTuple slices are smaller in size, they are still stored on the CERN grid and record low-level system information, such as tagging scores, PDF variations, etc. For angular analysis, researchers need to identify the particle flavors and reconstruct the high-level kinematic quantities (e.g., scattering angles).

The **NTupleRecon** repository is designed to download the NTuple slices from the **CERN grid** to the **lxplus** server, reconstruct the physical quantities of interest, discard the low-level data, and merge the files. In addition, metadata such as total events, cross sections, and generator filter efficiencies are automatically retrieved using *AMI* and updated to the Google Sheet, helping with data managements.

## Dependencies
The code is designed to run on the CERN lxplus machine. Before the first running, install the necessary packages for python scripts:
```bash
    pip install -r ~/.../ntuplerecon/dependencies.txt
```
Every time before running, set up the lxplus environment by ```source```:
```bash
    source ~/.../setup_env.sh
```
It mainly sets up environments working for ```python```, ```rucio```, ```pyami```.

## Repository Strucutre
- *AngleRecon/looper.cpp*: Main program for reconstructing physics process.
- *AngleRecon/reco_controller.py*: Controller of the ```looper```, responsible for iterating over NTuple slices within a given signal directory and reconstructing the angular variables.
- *DataFetch/datalist.py*: Stores the link to the Google Sheet, and the metadata of the real data and Monte Carlo samples.
- *DataFetch/datafetch.py*: **Core controller** that automates the process of downloading, reconstruction, and merging of datasets.
```
    ./DataFetch/datafetch.py
        -> ./AngleRecon/reco_controller.py
            -> ./AngleRecon/build/looper (C++ executable)
```
- *Draw/DataList.py*: Utility script that collects reconstructed file paths. Used by drawing scripts.
- *Draw/EFT_sensitivity.py*/*Draw/EFT_sensitivity_matplotlib.py*: Scripts for generating 1D distributions of physics observables with different EFT reweights.
- *SignalControl.py*: Produces control plots for Monte Carlo validation against data.
```
    ntuplerecon/

    | -- dependencies.txt
    | -- setup_env.sh
    | -- config.json

    | -- AngleRecon/
         | -- reco_controller.py
         | -- looper.cpp
         | -- CMakeLists.txt

         | -- build/
         | -- | -- looper (C++ executable)

    | -- DataFetch/
    |    | -- datafetch.py
         | -- datalist.py
    
    | -- Draw/
         | -- DataList.py
         | -- EFT_sensitivity.py
         | -- EFT_sensitivity_matplotlib.py
         | -- SignalControl.py
    
    | -- README.md
    | -- .gitignore
```

## Usage

Set up ```datalist.py``` and ```config.json``` properly.
- Specify the *spreadsheet link* and *metadata* in ```datalist.py```.
- Sepcify the *EFT reweighting points*, *work directory*, *real data path*, and *sensitivity plot directory* in ```config.json```.

Set up the environment on the lxplus machine
```bash
    source setup_env.sh
```

### Data Processing
In ```datafetch.py``` script, set the signal source to be processed, take ```ttbarEFT``` as an example, through setting 

```
rucio_reco_merge(datalist.ttbarEFT)
```

And if processing the real data from RUN2, and the data haven't been downloaded yet, set 

```
rucio_reco_merge(datalist.RealDataAll, download = True, real_data=True)
``` 

So that the ```rucio_reco_merge``` will call ```rucio``` to download data from the Grid to the lxplus server. The ```real_data=True``` setting is necessary, by which the program will skip looking for Monte Carlo reweights. By default, ```download``` and ```real_data``` are set to ```False```.

Update the Google Sheet with the general data information, such as *Total Events*, *Cross Section*, *Generator Filter Efficiency*, and *KFactor*, by setting ```updateGoogleSheet(datalist.spreadsheet)``` in the script.

![Example of Google Sheet Record](/assets/img/PostImages/GoogleRecord.png){: w="500" h="500" }
_Google Sheet Record Example_

With all these settings done in the script, just simply call
```bash
    python ~/.../ntuplerecon/DataFetch/datafetch.py
```
Then the automated data processing flow will begin. 

![Data Process Workflow](/assets/img/PostImages/DataWorkFlow.png){: w="500" h="500" }
_Data Process Workflow_

The Google Sheet recordings will be updated accordingly, as demonstrated below.

![Google Sheet Update](/assets/img/PostImages/datafetch_demo.gif){: w="500" h="500" }
_Automated Google Sheet Update_

To generate the signal control plots, first ensure that the configuration files and environment are set up as described in the previous sections. Then specify the physics observables to be plotted in the script ( for example, $$cos(\theta^+)$$ )
```
    draw_SignalControl("cos_theta_plus")
```
Run the script by executing
```bash
    python ~/.../ntuplerecon/Draw/SignalControl.py
```

To generate the EFT sensitivity plots, define the desired physics observables ( for example, $$cos(\theta^+)$$ )
```
    draw_EFTsensitivity("cos_theta_COM")
```
Execute the script with:
```bash
    python ~/.../ntuplerecon/Draw/EFT_sensitivity.py
```

### ⚠️ Note:
```EFT_sensitivity.py``` uses ROOT for plotting, which is slow due to the design of the ROOT library. It is recommended to first test and validate plots using ```EFT_sensitivity_matplotlib.py``` before running the ROOT-based script.

## Remarks
*There could be error messages during ```hadd``` merge, just ignore it. It doesn't interrupt the merging process. No direct evidence found that the error messages affect the final merging results.*

## 🚧 Project Status
This physics analysis project is currently under active development. There would be modifications on the reconstruction looper, the drawing scripts, and the systematics analysis updates.