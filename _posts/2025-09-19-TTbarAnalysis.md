---
title: Analysis of Top Quark Pair Decay
math: true
permalink: /Analysis Project/
---

## Overview
In the Beyond Standard Model (BSM) analysis, the top quark has large mass ($$m_t \approx 173~GeV$$) and short lifetime ($$O(10^{-25})$$s). The top quarks decay before hadronization. This makes the precise measurement of top quark properties possible, allowing stringent tests of the Standard Model (SM). We are interested in the $t\bar{t}$ decay system due to its abundant event statistics at the LHC and its characteristic final-state topologies.

In BSM, the effective Lagrangian can be written as

$$\mathcal{L}_{\text{eff}} = \mathcal{L}_{\text{SM}}^{(4)} + \frac{1}{\Lambda} \sum_i c_i^{(5)} \mathcal{O}_i^{(5)}
+ \frac{1}{\Lambda^2} \sum_i c_i^{(6)} \mathcal{O}_i^{(6)} + O\left( \frac{1}{\Lambda^3} \right)$$

where $$\Lambda$$ is the energy scale at which the EFT is not applicable. The $$\mathcal{O}_i^{(x)}$$ refers to dimension-$x$ operators and $$c^{(x)}_i$$ are the Wilson coefficients. 

The goal of this project is to **constrain the Wilson coefficients** contributing to top quark decay, **perform hypothesis tests** on the Standard Model, thus finding the evidence of New Physics.

![Feynmann Diagram of Top Pair Decays](/assets/img/PostImages/ttbarFeynmann.png){: w="200" h="200" }
_Feynmann Diagram of Top Pair Decays_

## Simulation

In order to study the beyond standard model behaviors and perform hypothesis tests on the standard model, high quality simulation on the particle decaying process is necessary. The Monte Carlo simulation in CERN is mainly made up of these steps: *Generation*, *Detector Simulation*, *Reconstruction*, *Derivation*.

- **Generation**: Theory-driven simulation of parton-level events via cross-section calculations. The resulting partons (quarks, gluons) then go through the parton showering and hadronisation process, forming hadronic states due to the QCD radiation. The tools frequently used are *MadGraph*, *Powheg*, *Sherpa* and *Pythia*.

- **Detector Simulation**: Interacts the generated particles with the virtual detector model, using full (*GEANT4*) or fast (*AtlFast*) simulations. Produces the detector cell energy deposits stored in the HITS files. Our group is concentrating on the development of [**GeoModel**](/GeoModel/), a visualization toolkit for detector simulation.


![Detector Level Simulation](/assets/img/PostImages/DetectSimu.png){: w="400" h="400" }
_Detector Level Simulation with GeoModel Toolkit_

- **Reconstruction**: Transforms detector signals into high-level physics objects (jets, leptons, missing energy, etc.) through pile-up, digitization, trigger, and reconstruction algorithms. Results are stored in AOD files.

- **Derivation**: Keep the data only relavent to the specific analysis, throwing low-quality events, low-level detector simulation information, etc. Results are stored in DAOD files used for physics analysis.

In the early stage of this project, we conducted generator-level tests using *MadGraph*. The study shows that $$t\bar{t}$$ system is unpolarized while the spins are highly correlated, which proves that our simulation settings are correct.

![Generator Level Study](/assets/img/PostImages/MadGraph.png){: w="300" h="300" }
_Generator Level Study with MadGraph_

## Event Skimming
The [TopCPToolkit](https://topcptoolkit.docs.cern.ch/latest/) is used for **event skimming** -- only events passed selection barriers are kept, while discarding those with low quality particles (e.g., low $$p_T$$ objects), or failing flavor tagging algorithms. This procedure dramatically reduces the data from **hundreds of terabytes** to only **hundreds of gigabytes**. Since the DAOD files generated from the Monte Carlo simulation workflow are distributed across worldwide computing clusters, the skimming jobs are executed on the clusters and their progress can be monitored through [PanDA](https://panda-wms.readthedocs.io/en/latest/index.html#).

![PanDA Monitor](/assets/img/PostImages/PanDA.png){: w="250" h="250" }
_PanDA Monitor with Jobs_

Event skimming is applied on all MC samples contributing to the $$t\bar{t}$$ signal as well as the $$W+$$jets background that can be misidentified as $$t\bar{t}$$. The details of these tasks—including task IDs, input datasets, and associated metadata—are systematically recorded in a shared Google Sheet. The resulting files after skimming are **NTuple** files, a data format compatible with the [ROOT framework](https://root.cern/). Up to now, the NTuple files contain extensive system-level information, such as tagging scores and PDF variations, and low-level physics observables, including missing transverse energy (not exactly the nutrino energy), lepton and jet directions.

## Reconstruction and Slimming
The **reconstruction process** is supposed to infer the flavor of jets, reconstruct the 4-momenta of the inital/intermediate/final state particles, and compute higher-level physics observables related to the analysis. Then the **slimming process** removes most of the system-level information, and retains the reconstructed 4-momena of the full physics process, together with higher-level observables. These are stored in **ROOT** files, efficient for further analysis.

I developed the [**NTupleRecon**](https://gitlab.cern.ch/atlas-ttbar-10-angle/ntuplerecon/) repository to process the NTuple files generated by TopCPToolkit. The repository is designed to automate all steps following event skimming, providing an integrated workflow to transform raw NTuples into analysis-ready data. The main steps are:
- **Download**: Retrieve NTuple slices from the grid to the lxplus server using ```rucio```.
- **Reconstruct and Slim**: Reconstruct high-level physics observables and slim the files using custom C++ algorithms, producing new ROOT slices.
- **Merge**: Combine all slices into a single ROOT file for subsequent analysis by calling ```hadd```.

The [workflow diagram](/assets/img/PostImages/DataWorkFlow.png) illustrates how the **mapping** and **reducing** steps are performed by TopCPToolkit and NTupleRecon in producing the ready-to-use data. 

More detailed usage of the NTupleRecon repository can be found [here](/NTuple Reconstruction/).

![Data Process Workflow](/assets/img/PostImages/DataWorkFlow.png){: w="500" h="500" }
_Data Process Workflow_

## Angular Analysis of $$t\bar{t}$$ System
With the ROOT files ready, we proceed to the angular analysis part of the $$t\bar{t}$$ system. The main idea is to use Fourier series coefficients as data features and perform minimum $$\chi^2$$ fittings to extract Wilson coefficients.
The $t\bar{t}$ decay angle distribution can be expressed in the M-function basis as

$$\rho (\theta^+, \phi^+, \theta^{*+}, \phi^{*+}, \theta^-, \phi^-, \theta^{*-}, \phi^{*-}) =
\sum_{\{j_n\}, \{m_n\}} c_{m_1 m_2 m_3 m_4}^{j_1 j_2 j_3 j_4} M_{m_1 m_2}^{j_1 j_2} (\phi^+, \theta^+, \phi^{*+}, \theta^{*+})  M_{m_3 m_4}^{j_3 j_4} (\phi^-, \theta^-, \phi^{*-}, \theta^{*-})$$

the angles are the azimuthal and polar angles from (anti)top quark decay and (anti)W boson decay. Using the orthonormal characteristics of M-function, the series expansion coefficients can be extracted by

$$c_{m_1 m_2 m_3 m_4}^{j_1 j_2 j_3 j_4} = \int d\Omega d\Omega^* \, \rho (\boldsymbol{\vec{\theta}^+}, \boldsymbol{\vec{\theta}^-}) (M_{m_1 m_2}^{j_1 j_2} (\boldsymbol{\vec{\theta}^+}) \cdot M_{m_3 m_4}^{j_3 j_4} (\boldsymbol{\vec{\theta}^-}) )^*$$

$\boldsymbol{\vec{\theta}^{+(-)}}$ denotes $$ \{ \phi^{+(-)}, \theta^{+(-)}, \phi^{*+(-)}, \theta^{*+(-)} \} $$. With those features extracted from the high-dimension distribution, we can construct a measurement between the BSM events and the SM events

$$\chi^2 = (\boldsymbol{c}_{BSM}(\boldsymbol{\vec{\alpha}}) -\boldsymbol{c}_{SM})^T(Cov^{BSM} + Cov^{SM})^{-1}(\boldsymbol{c}_{BSM}(\boldsymbol{\vec{\alpha}}) -\boldsymbol{c}_{SM})$$

and determine confidence intervals on the Wilson coefficients $\boldsymbol{\vec{\alpha}}$.

## Analysis Framework

I developed the [tenAngleAnalysis](https://gitlab.cern.ch/ruxue/tenangleanalysis) repository for the angular series analysis works. As shown in the [structure](/assets/img/PostImages/tenAngleAnalysis.png), the repository takes the processed ROOT data as inputs, and the repository is primarily composed of the ```AngleAnalyze``` class, performing MM-function decomposition, and the ```DBHandler``` class, managing data storaging and fetching. Researchers can extend the framework by developing their own *.cpp* or *.py* scripts for more advanced numerical studies, such as morphing, error analysis, and dimension projections. More detailed usage of the framework can be found [here](/Ten Angle Analysis/).

![Repository Structure](/assets/img/PostImages/tenAngleAnalysis.png){: w="400" h="400" }
_Repository Structure_

Consistensy checks are also performed to validate the repository in case of any coding bugs. The idea is reconstructing the decay distribution $$\rho(\boldsymbol{\vec{\theta}^+}, \boldsymbol{\vec{\theta}^-})$$ using the extracted series coefficients, projecting reconstructed distribution into one-dimensional subspaces, and comparing the results with the original simulation output. The results demonstrate that the model behaves as expected and that the framework contains no apparent implementation issues.

![Validation Results](/assets/img/PostImages/validation.png){: w="400" h="400" }
_Validation Results_

## Future Works

The story is still ongoing, with many obstacles waiting to be conquered. Additional phenomenological observables and their associated orthogonal functions, implementing realistic error analysis under systematic variations, robust and high-efficiency morphing methods are on the to-do list of this project. Ultimately, our goal is to apply these techniques to real data collected at CERN in the search for evidence of new physics.

👉Stay tunned and wish us the best luck!🍀🚀