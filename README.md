# CMSSW

## Overview
The overall collection of software, referred to as CMS Software (CMSSW), is built around a Framework, an Event Data Model (EDM), and Services needed by the simulation, calibration and alignment, and reconstruction modules that process event data so that physicists can perform analysis. The primary goal of the Framework and EDM is to facilitate the development and deployment of reconstruction and analysis software.

The CMSSW event processing model consists of one executable, called `cmsRun`, and many plug-in modules which are managed by the Framework. All the code needed in the event processing (calibration, reconstruction algorithms, etc.) is contained in the modules. The same executable is used for both detector and Monte Carlo data. More and detailed information can be found [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCMSSWFramework). 



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
   - **mkrecord :** makes a complete implementation of a Record used by the [EventSetup](https://twiki.cern.ch/twiki/bin/view/Main/CMSSWatFNALFramework#Module_types)
   
More generators are available and you can find them [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideSkeletonCodeGenerator) 


## Configure

### Set up your Environment
If you have an account on *lxplus* or *cmslpc* clusters, you can login one of these clusters using *ssh* command.
 - ssh -Y [yourusername]@lxplus.cern.ch  
 *make a your working directory*

```javascript
 mkdir MYDEMOANALYZER
 cd MYDEMOANALYZER
```

```html
# if output of echo $0 is csh or tcsh
   setenv SCRAM_ARCH slc6_amd64_gcc491

# if output of echo $0 is bash/sh
   export SCRAM_ARCH=slc6_amd64_gcc491

# check your arch - it should give an output as slc6_amd64_gcc491
   echo $SCRAM_ARCH
```
```javascript
 # create a new project area 
 cmsrel CMSSW_7_4_15 
 echo $0

 cd CMSSW_7_4_15/src/ 
 cmsenv
 ```

### Write a Framework Module

First, create a subsystem area. The actual name used for the directory is not important, we'll use Demo. For more information click [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookWriteFrameworkModule). From the src directory, make and change to the Demo area:

```javascript
mkdir Demo
cd Demo
```
Note that if you do not create the subsystem area and create you module directly under the src directory, your code will not compile. Create the "skeleton" of an EDAnalyzer module (see [SWGuideSkeletonCodeGenerator for more information](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideSkeletonCodeGenerator)):

```javascript
mkedanlzr DemoAnalyzer
```
Compile the code:

```javascript
cd DemoAnalyzer
scram b
```
We'll use a ROOT file as the data source when we run with this module. The data source is defined in the configuration file. For your convenience, there is already a data file processed in **CMSSW_5_3_4**, which is fully compatible with **CMSSW_7_4_15** release, containing 100 events. It is located at `/afs/cern.ch/cms/Tutorials/TWIKI_DATA/TTJets_8TeV_53X.root`.

The **mkedanlzr** script has generated an example python configuration file ConfFile_cfg.py in the **~DemoAnalyzer/python** directory. Open the file using your favorite text editor and change the data source file to as shown below.

And now the configuration file should read like this:

```javascript
import FWCore.ParameterSet.Config as cms

process = cms.Process("Demo")

process.load("FWCore.MessageService.MessageLogger_cfi")

process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(-1) )

process.source = cms.Source("PoolSource",
                                # replace 'myfile.root' with the source file you want to use
                                fileNames = cms.untracked.vstring(
            'file:/afs/cern.ch/cms/Tutorials/TWIKI_DATA/TTJets_8TeV_53X.root'
                )
                            )

process.demo = cms.EDAnalyzer('DemoAnalyzer'
                              )

process.p = cms.Path(process.demo)
```

Full documentation about the configuration language is in [SWGuideAboutPythonConfigFile](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideAboutPythonConfigFile).

Run the job

```javascript
cmsRun Demo/DemoAnalyzer/python/ConfFile_cfg.py
```

### Get tracks from the Event
`Demo/DemoAnalyzer/plugins/DemoAnalyzer.cc` is the place to put your analysis code. In this example, we only add very simple statement printing the number of tracks.

Edit `Demo/DemoAnalyzer/plugins/BuildFile.xml:` so that it looks like this
```javascript
<use name="FWCore/Framework"/>
<use name="FWCore/PluginManager"/>
<use name="FWCore/ParameterSet"/>
<use name="DataFormats/TrackReco"/>
<use name="CommonTools/UtilAlgos"/>
<flags EDM_PLUGIN="1"/>
```
More on information on the structure of a typical `BuildFile.xml` can be found on [WorkBookBuildFilesIntro](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookBuildFilesIntro).

Edit `Demo/DemoAnalyzer/plugins/DemoAnalyzer.cc`
 - Add the following include statements (together with the other include statements):

```javascript
#include "DataFormats/TrackReco/interface/Track.h"
#include "DataFormats/TrackReco/interface/TrackFwd.h"
#include "FWCore/MessageLogger/interface/MessageLogger.h"
```
- Edit the method `analyze` which starts with

```javascript
DemoAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup)
```
and put the following lines below `using namespace edm;`

```javascript
Handle<reco::TrackCollection> tracks; 
iEvent.getByLabel("generalTracks", tracks);
LogInfo ("Demo") << "number of tracks " << tracks->size();
```
To see how to access data from a triggered or simulated physics event look at [SWGuideEDMGetDataFromEvent](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideEDMGetDataFromEvent).

To know what other collections (besides `TrackCollection` of reconstructed objects are available, you can have a look to [SWGuideRecoDataTable](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideRecoDataTable).

Now, compile the code and run the job again:

 ```javascript
 scram b
 cmsRun Demo/DemoAnalyzer/python/ConfFile_cfg.py  
  ```
The output should look something like this:

 ```javascript
 [lxplus404 @ ~/workbook/MYDEMOANALYZER/CMSSW_5_3_5/src]$  cmsRun Demo/DemoAnalyzer/demoanalyzer_cfg.py
12-Mar-2013 18:59:31 CET  Initiating request to open file file:/afs/cern.ch/cms/Tutorials/TTJets_RECO_5_3_4.root
12-Mar-2013 18:59:36 CET  Successfully opened file file:/afs/cern.ch/cms/Tutorials/TTJets_RECO_5_3_4.root
%MSG-i Root_Information:  AfterFile TClass::TClass() 12-Mar-2013 18:59:36 CET  pre-events
no dictionary for class pair<edm::IndexIntoFile::IndexRunKey,Long64_t> is available
%MSG
%MSG-i Root_Information:  AfterFile TClass::TClass() 12-Mar-2013 18:59:36 CET  pre-events
no dictionary for class pair<edm::IndexIntoFile::IndexRunLumiKey,Long64_t> is available
%MSG
%MSG-i Root_Information:  AfterFile TClass::TClass() 12-Mar-2013 18:59:36 CET  pre-events
no dictionary for class pair<edm::BranchKey,edm::ConstBranchDescription> is available
%MSG
%MSG-i Root_Information:  AfterFile TClass::TClass() 12-Mar-2013 18:59:36 CET  pre-events
no dictionary for class pair<edm::BranchID,unsigned int> is available
%MSG
Begin processing the 1st record. Run 1, Event 261746003, LumiSection 872662 at 12-Mar-2013 18:59:37.193 CET
%MSG-i Demo:  DemoAnalyzer:demo 12-Mar-2013 18:59:37 CET Run: 1 Event: 261746003
number of tracks 1211
%MSG
Begin processing the 2nd record. Run 1, Event 261746009, LumiSection 872662 at 12-Mar-2013 18:59:37.203 CET
%MSG-i Demo:  DemoAnalyzer:demo 12-Mar-2013 18:59:37 CET Run: 1 Event: 261746009
number of tracks 781
%MSG
Begin processing the 3rd record. Run 1, Event 261746010, LumiSection 872662 at 12-Mar-2013 18:59:37.206 CET
%MSG-i Demo:  DemoAnalyzer:demo 12-Mar-2013 18:59:37 CET Run: 1 Event: 261746010
number of tracks 1535
...
%MSG
Begin processing the 48th record. Run 1, Event 261746140, LumiSection 872662 at 12-Mar-2013 18:59:38.129 CET
%MSG-i Demo:  DemoAnalyzer:demo 12-Mar-2013 18:59:38 CET Run: 1 Event: 261746140
number of tracks 544
%MSG
Begin processing the 49th record. Run 1, Event 261746141, LumiSection 872662 at 12-Mar-2013 18:59:38.134 CET
%MSG-i Demo:  DemoAnalyzer:demo 12-Mar-2013 18:59:38 CET Run: 1 Event: 261746141
number of tracks 662
%MSG
Begin processing the 50th record. Run 1, Event 261746142, LumiSection 872662 at 12-Mar-2013 18:59:38.135 CET
%MSG-i Demo:  DemoAnalyzer:demo 12-Mar-2013 18:59:38 CET Run: 1 Event: 261746142
number of tracks 947
%MSG
12-Mar-2013 18:59:38 CET  Closed file file:/afs/cern.ch/cms/Tutorials/TTJets_RECO_5_3_4.root
TrigReport ---------- Event  Summary ------------
TrigReport Events total = 50 passed = 50 failed = 0

TrigReport ---------- Path   Summary ------------
TrigReport  Trig Bit#        Run     Passed     Failed      Error Name
TrigReport     1    0         50         50          0          0 p

TrigReport -------End-Path   Summary ------------
TrigReport  Trig Bit#        Run     Passed     Failed      Error Name

TrigReport ---------- Modules in Path: p ------------
TrigReport  Trig Bit#    Visited     Passed     Failed      Error Name
TrigReport     1    0         50         50          0          0 demo

TrigReport ---------- Module Summary ------------
TrigReport    Visited        Run     Passed     Failed      Error Name
TrigReport         50         50         50          0          0 demo
TrigReport         50         50         50          0          0 TriggerResults

TimeReport ---------- Event  Summary ---[sec]----
TimeReport CPU/event = 0.002820 Real/event = 0.002912

TimeReport ---------- Path   Summary ---[sec]----
TimeReport             per event          per path-run 
TimeReport        CPU       Real        CPU       Real Name
TimeReport   0.002800   0.002883   0.002800   0.002883 p
TimeReport        CPU       Real        CPU       Real Name
TimeReport             per event          per path-run 

TimeReport -------End-Path   Summary ---[sec]----
TimeReport             per event       per endpath-run 
TimeReport        CPU       Real        CPU       Real Name
TimeReport        CPU       Real        CPU       Real Name
TimeReport             per event       per endpath-run 

TimeReport ---------- Modules in Path: p ---[sec]----
TimeReport             per event      per module-visit 
TimeReport        CPU       Real        CPU       Real Name
TimeReport   0.002800   0.002881   0.002800   0.002881 demo
TimeReport        CPU       Real        CPU       Real Name
TimeReport             per event      per module-visit 
TimeReport        CPU       Real        CPU       Real Name
TimeReport             per event      per module-visit 

TimeReport ---------- Module Summary ---[sec]----
TimeReport             per event        per module-run      per module-visit 
TimeReport        CPU       Real        CPU       Real        CPU       Real Name
TimeReport   0.002800   0.002881   0.002800   0.002881   0.002800   0.002881 demo
TimeReport   0.000020   0.000025   0.000020   0.000025   0.000020   0.000025 TriggerResults
TimeReport        CPU       Real        CPU       Real        CPU       Real Name
TimeReport             per event        per module-run      per module-visit 

T---Report end!

=============================================

MessageLogger Summary

 type     category        sev    module        subroutine        count    total
 ---- -------------------- -- ---------------- ----------------  -----    -----
    1 fileAction           -s file_close                             1        1
    2 fileAction           -s file_open                              2        2

 type    category    Examples: run/evt        run/evt          run/evt
 ---- -------------------- ---------------- ---------------- ----------------
    1 fileAction           PostEndRun                        
    2 fileAction           pre-events       pre-events       

Severity    # Occurrences   Total Occurrences
--------    -------------   -----------------
System                  3                   3  
   ```





## Conditions Data




#### Examples  
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
