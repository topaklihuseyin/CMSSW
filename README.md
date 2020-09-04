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

Unlike the previous event processing frameworks, cmsRun is extremely lightweight: only the required modules are dynamically loaded at the beginning of the job.




## Data Model


## Analyzers


## Configure


## Conditions Data



- First Title
- Second Title
- Third Title
 - indented
   1. inner number
   2. outer number
   3. three
   
   [This is the descrition](https://github.com/topaklihuseyin/CMSSW/edit/master/README.md)
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
