# Radar Aeroecology Virtual Research Environment

## Table of contents
1. [Introduction](#introduction)
    1. [Passerines](#passerines)
    2. [Meteorological Radar](#meteorological-radar)
    3. [Approach](#approach)
2. [Exercises](#exercises)
    1. [Introduction](#introduction-1)
    2. [Exercise 1. Configuration](#exercise-1-configuration)
    3. [Exercise 2. Accessing KNMI Data Platform](#exercise-2-accessing-knmi-data-platform)


## Introduction
### Passerines
Small passerine birds migrate en masse during Spring and Fall. These migration events can be significant enough to observe on Meteorological Radars. Passerine birds do not migrate continuously throughout the season but instead seemingly wait for favorable weather conditions. These weather conditions have been identified to a certain extend and as such certain moments can be defined on which peak migration would likely occur. 
### Meteorological Radar
Meteorological radars monitor the sky 24/7 and are able to detect small objects, such as birds. A Meteorological radar emits a beam and attempts to detect the reflection of the beam. Most objects, including birds, tend to reflect at least part of the energy of the radar beam. This makes Meteorological radars ideal to use in observing peak migration events. However, as a Meteorological radar is not purposefully built to detect small passerines, some effort has to be made in order to produce meaningful data. Furthermore, as ideal as it sounds that a device measures 24/7 you will end up with more information than you generally can handle, some pruning is required. 
### Approach
As we have a decent understanding of favorable weather conditions we can start to narrow down our search towards the few times in the year that we have favorable conditions for passerines. Furthermore, we can apply certain transformations to the data which will allow us to filter, summarize and visualize the data which will help us focus on the actual migration rather than weather phenomena. 
### Exercises
#### Introduction
For this introduction, you will be introduced to some of the core principles of Radar Aeroecology, specifically Meteorological Radar data analysis. You will download Meteorological Radar Data from The Netherlands from multiple radars, process, visualize and interpret. You do not have to be proficient in either Python or R to complete these exercises. This introduction will be done in the Radar Aeroecology Virtual Research Environment (RAVRE) which you will be introduced to through exercises. After the introduction you will be guided to choose a moments in time to determine if you can witness a peak migration event. Furthermore, you will be introduced to more advanced visualization techniques in R by using the `bioRad` [paper](https://onlinelibrary.wiley.com/doi/10.1111/ecog.04028), [library documentation](https://adriaandokter.com/bioRad/articles/bioRad.html) package.
RAVRE is a VRE built on NaaVRE. That means that RAVRE follows the guidelines of NaaVRE which means there is a specific way of doing things. Throughout these exercises you will be asked to copy and paste code into jupyter cells. 
Most exercises have a `Note:` section. This section generally explains a bit more about the how and what about your copy and paste. This information is not necessary to go through the introduction but might be interesting to read. 

##### Exercise 1. Configuration
First, we need to set certain parameters and configurations. We will be using a public API ([What is API?](https://www.ibm.com/topics/api)) key from the Dutch meteorological institute, KNMI (Koninklijk Nederlands Meteorologisch Instituut). With this key, we can authenticate ourselves within the data repository of [The KNMI Data Platform](https://dataplatform.knmi.nl/). From this platform, we will be retrieving Polar Volume Data which contains a host of parameters which can be used to detect biological echoes. 
```python
# conf
import os
param_radar = "" # Den Helder, Herwijnen
param_api_key = "" #
param_start_date = "" # %Y%m%d%H%M ; 202010152355
param_end_date = "" # %Y%m%d%H%M ; 202010152355
conf_storage_dir = f"{os.environ.get("USER")}/data" # Set the storage directory to your personal directory/data.
if not os.path.exists(conf_storage_dir): # If this directory does not exist...
    os.path.mkdir(conf_storage_dir) # Create the directory
```
`Note:` There are two types of parameters recognized by NaaVRE Code analyzer: `param` and `conf`. The keyword `param` signifies that the user needs to supply this information when a workflow is executed. This could be a name of a Radar, a date, your password, etc. The keyword `conf` signifies that it holds information publicly shared. You would not enter your password here as it would be public to the world. You would use `conf` to indicate where your storage is located, what your maximum number of files needs to be, etc. 

Now we have the code inserted, you should be able to see the following:
`image of vre notebook with conf`

On the left hand side, you can see the `component containeriser`. `image of component containeriser`. This allows us to store the code we just entered in the notebook as a code-block. We will eventually make a few more of these code-blocks and use them to make a workflow out of them. Click the `component containeriser` and set `ABC to EFG` and the base image as vol2bird. Now your code cell is configured and you can go ahead and press `Create`. It will take a few minutes before it shows up in our cell catalogue. Instead of waiting, we are going to continue adding some more blocks.

##### Exercise 2. Accessing KNMI Data Platform
The next step is to create a code-block which can use this information we've specified and talk to the data repository of the KNMI. We are going to create a code-block which can ask the API which files it has for our search query. We will search by specifying three parameters: `start_date`, `end_date`, `radar`. The API will then return us (if successful) a list of file names corresponding to Radar Measurements for our query. 

```python
some code to list files
```
