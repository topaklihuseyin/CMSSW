# CMSSW

## Overview
The overall collection of software, referred to as CMSSW, is built around a Framework, an Event Data Model (EDM), and Services needed by the simulation, calibration and alignment, and reconstruction modules that process event data so that physicists can perform analysis. The primary goal of the Framework and EDM is to facilitate the development and deployment of reconstruction and analysis software.

The CMSSW event processing model consists of one executable, called cmsRun, and many plug-in modules which are managed by the Framework. All the code needed in the event processing (calibration, reconstruction algorithms, etc.) is contained in the modules. The same executable is used for both detector and Monte Carlo data.

The CMSSW executable, cmsRun, is configured at run time by the user's job-specific [configuration file](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookConfigFileIntro). This file tells cmsRun
- which data to use
- which modules to execute
- which parameter settings to use for each module
- what is the order or the executions of modules, called *path*
- how the events are filtered within each path, and
- how the paths are connected to the output files

cmsRun is extremely lightweight: only the required modules are dynamically loaded at the beginning of the job. More information can be found [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCMSSWFramework).

## Data Model
The CMS Event Data Model (EDM) is centered around the concept of an *Event*. Physically, an event is the result of a single readout of the detector electronics and the signals that will (in general) have been generated by particles, tracks, energy deposits, present in a number of bunch crossings. 

In software terms, an Event starts as a collection of the RAW data from a detector or MC event, stored as a single entity in memory, a C++ type-safe container called edm::Event. An Event is a C++ object container for all RAW and reconstructed data related to a particular collision. During processing, data are passed from one module to the next via the Event, and are accessed only through the Event. All objects in the Event may be individually or collectively stored in ROOT files, and are thus directly browsable in ROOT. This allows tests to be run on individual modules in isolation. Auxiliary information needed to process an Event is called Event Setup, and is accessed via the EventSetup.

#### The CMS Data Hierarchy 
CMS Data is arranged into a hierarchy of data tiers. Each physics event is written into each data tier, where the tiers each contain different levels of information about the event. The different tiers each have different uses. The three main data tiers written in CMS are:
1. RAW: full event information from the Tier-0 (i.e. from CERN), containing 'raw' detector information (detector element hits, etc)
   - RAW is not used directly for analysis
   
2. RECO ("RECOnstructed data"): the output from first-pass processing by the Tier-0. This layer contains reconstructed physics objects, but it's still very detailed. 
    - RECO can be used for analysis, but is too big for frequent or heavy use when CMS has collected a substantial data sample.
    
3. AOD ("Analysis Object Data"): this is a "distilled" version of the RECO event information, and is expected to be used for most analyses.
   - AOD provides a trade-off between event size and complexity of the available information to optimize flexibility and speed for analyses.

The data tiers are described in more detail in a dedicated WorkBook chapter on [Data Formats and Tiers](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookDataFormats). 

#### Detector data flow through Hardware Tiers
The following diagram shows the flow of CMS detector data through the tiers.
![alt text](https://twiki.cern.ch/twiki/pub/CMSPublic/WorkBookComputingModel/dataflowtiers_Data.gif)
The essential elements of the flow of real physics data through the hardware tiers are:
- T0 to T1:
   - Scheduled, time-critical, will be continuous during data-taking periods
   - reliable transfer needed for fast access to new data, and to ensure that data is stored safely
- T1 to T1:
     - redistributing data, generally after reprocessing (e.g. processing with improved algorithms)
- T1 to T2:
    - Data for analysis at Tier-2s
    
 #### Monte Carlo data flow through Hardware Tiers
 Monte Carlo generated data is typically produced at a T2 center, and archived at its associated T1 and made available to the whole CMS collaboration. The following diagram shows the flow of Monte Carlo data through the tiers.
 ![alt text](https://twiki.cern.ch/twiki/pub/CMSPublic/WorkBookComputingModel/dataflowtiers_MC.gif)
 
    

## Analyzers
First, a few general words about analysis in the CMSSW framework. Physics analysis proceeds via a series of subsequent steps. Building blocks are identified and more complex objects are built on top of them. For instance, the Higgs search H ->ZZ -> µµµµ requires:
 - identifying muon candidates;
 - reconstructing Z candidates starting from muon candidates;
 - reconstructing Higgs candidates starting from Z candidates.
 
This process clearly identifies three products: muon candidates, Z candidates, and a Higgs candidate, as well as three processes to reconstruct them. These are well mapped into three Framework modules (EDProducers) that add into the Event three different products (the candidates collections). For more information click [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookWriteFrameworkModule).

When setting up code for the new EDM (such as creating a new EDProducer) there is a fair amount of 'boiler plate' code that you must write. To make writing such code easier CMS provides a series of scripts that will generate the necessary directory structure and files needed so that all you need to do is write your actual algorithms.

CMSSW distiguishes following [module types](https://twiki.cern.ch/twiki/bin/view/Main/CMSSWatFNALFramework#Module_types):

 - **EDAnalyzer:** takes input from the event and processes the input without writing information back to the event
 - **EDProducer:** takes input from the event and produces new output which is saved in the event
 - **EDFilter:** decides if processing the event can be stopped and continued
 - **EventSetup:** external service not bound to the event structure which provides information useable by all modules (e.g. Geometry, Magnetic Field, etc.)
  
  In order to generate above modules:
     - **mkedanlzr :** makes a skeleton of a package containing an [EDAnalyzer](https://twiki.cern.ch/twiki/bin/view/Main/CMSSWatFNALFramework#Module_types) 
   - **mkedprod :** makes a skeleton of a package containing an [EDProducer](https://twiki.cern.ch/twiki/bin/view/Main/CMSSWatFNALFramework#Module_types)  
  - **mkedfltr :** makes a skeleton of a package containing an [EDFilter](https://twiki.cern.ch/twiki/bin/view/Main/CMSSWatFNALFramework#Module_types)
  -  **mkrecord :** makes a complete implementation of a Record used by the [EventSetup](https://twiki.cern.ch/twiki/bin/view/Main/CMSSWatFNALFramework#Module_types)
 
## Configure


## Conditions Data



### Examples  
- First Title
 - indented
   1. inner number
      This paragraph has some `variable` inline code.
   
   ```html
   <p align="left">This is a paragraph example but p should not see on the letter but it seen.</p> 
   ```
   ```javascript
   let num =Math.random();
   ```
   ![alt text](http://picsum.photos/200/200)
   >block quote text below the paragraf
   
  | heading | header | head |
  | --- | --- | --- |
  | content | more content | text |
  | more | more | more |
 
 This being  *created* on a **Friday** ~~Saturday~~.
