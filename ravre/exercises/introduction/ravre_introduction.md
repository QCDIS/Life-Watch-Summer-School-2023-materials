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
    4. [Exercise 3. Downloading Polar Volume data](#exercise-3-downloading-polar-volume-data)
    5. [Exercise 4. Converting Polar Volume data](#exercise-4-converting-polar-volume-data)


## Introduction
### Passerines
Small passerine birds migrate en masse during Spring and Fall. These migration events can be significant enough to observe on Meteorological Radars. Passerine birds do not migrate continuously throughout the season but instead seemingly wait for favorable weather conditions. These weather conditions have been identified to a certain extend and as such certain moments can be defined on which peak migration would likely occur. 
### Meteorological Radar
Meteorological radars monitor the sky 24/7 and are able to detect small objects, such as birds. A Meteorological radar emits a beam and attempts to detect the reflection of the beam. Most objects, including birds, tend to reflect at least part of the energy of the radar beam. This makes Meteorological radars ideal to use in observing peak migration events. However, as a Meteorological radar is not purposefully built to detect small passerines, some effort has to be made in order to produce meaningful data. Furthermore, as ideal as it sounds that a device measures 24/7 you will end up with more information than you generally can handle, some pruning is required. 
### Approach
As we have a decent understanding of favorable weather conditions we can start to narrow down our search towards the few times in the year that we have favorable conditions for passerines. Furthermore, we can apply certain transformations to the data which will allow us to filter, summarize and visualize the data which will help us focus on the actual migration rather than weather phenomena. 
### Exercises
#### Introduction
For this introduction, you will be introduced to some of the core principles of Radar Aeroecology, specifically Meteorological Radar data analysis. You will download Meteorological Radar Data from The Netherlands from multiple radars, process, visualize and interpret. You do not have to be proficient in either Python or R to complete these exercises. This introduction will be done in the Radar Aeroecology Virtual Research Environment (RAVRE) which you will be introduced to through exercises. After the introduction you will be guided to choose a moments in time to determine if you can witness a peak migration event. Furthermore, you will be introduced to more advanced visualization techniques in R by using the `bioRad` ([paper](https://onlinelibrary.wiley.com/doi/10.1111/ecog.04028), [library documentation](https://adriaandokter.com/bioRad/articles/bioRad.html)) package.
RAVRE is a VRE built on NaaVRE. That means that RAVRE follows the guidelines of NaaVRE which means there is a specific way of doing things. Throughout these exercises you will be asked to copy and paste code into jupyter cells. 
Most exercises have a `Note:` section. This section generally explains a bit more about the how and what about your copy and paste. This information is not necessary to go through the introduction but might be interesting to read. 

##### Exercise 1. Configuration
First, we need to set certain parameters and configurations. We will be using a public API ([What is API?](https://www.ibm.com/topics/api)) key from the Dutch meteorological institute, KNMI (Koninklijk Nederlands Meteorologisch Instituut). With this key, we can authenticate ourselves within the data repository of [The KNMI Data Platform](https://dataplatform.knmi.nl/). From this platform, we will be retrieving Polar Volume Data which contains a host of parameters which can be used to detect biological echoes. 

`Important to know`: The API key used during this course is only valid for the duration of the course. KNMI has been kind and provided us with our own custom workshop API-key. They have done this to help us 'guarantee' to have access to the data without having to register or share the key with other anonymous users. As this is courtesy, please keep in mind to keep your queries short. Practically this means: Choose a day to retrieve data for, not a year. If you decide you want to see an effect over a large temporal time span, sample interval can be reduced. For example: every 15th day of the month, every X'th hour of the day. You do not have enough time during this course to download, convert and analyze large amounts of data nor will the API-key allow all students to download a large dataset. Rest assured, throughout the exercises you will be guided on how to query and stay within the `fair-use policy` and still get interesting results.  
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
The next step is to create a code-block which can use this information we've specified and communicate with the data repository of the KNMI. We are going to create a code-block which can ask the API which files it has for our search query. We will search by specifying three parameters: `start_date`, `end_date`, `radar`. The API will then return us (if successful) a list of file names corresponding to Radar Measurements for our query. 

```python
some code to list files
```
`Note:` The measurement interval for Dutch Meteorological Radars is 5 minutes. This means that each Radar can produce up to 288 measurements in a given day. Occasionally, radars fail to produce measurements and will therefore have less than 288 files.

Let's go ahead and request this code cell to be turned in a functional block, like we did with the previous exercise. Use the `component containeriser` to containerise the newly added code. Ensure to specify each of the types of variables correctly and to use the vol2bird base image. 

##### Exercise 3. Downloading Polar Volume data
Now we will be adding a block of code which can use the results from the previous block, files within our query, to download and store them at our convenience. Copy the code below and place it in your Jupyter notebook. Follow the same steps as previous exercises and containerise the code by using the `code containeriser` on your left.

```python
some code that downloads the listed files
```
`Note:`

##### Exercise 4. Converting Polar Volume data
The files that we are downloading are HDF5 files which contain Polar Volumetric data. The data is currently structured according the the [data model of KNMI](https://www.knmi.nl/kennis-en-datacentrum/publicatie/knmi-hdf5-data-format-specification-v3-5). Most of our analysis methods expect a data model of [ODIM](https://www.eumetnet.eu/wp-content/uploads/2019/01/ODIM_H5_v23.pdf) which means we need to run a converter. The following code expects a list of files and will call a converter to convert the KNMI format to an ODIM format and returns us a list of files in ODIM format. 
```python
code that calls KNMI converter
```
`Note:` Both the KNMI and ODIM use the same filetype, [HDF5](https://www.hdfgroup.org/solutions/hdf5/). HDF5 is an industry standard in self describing data storage. It's been around for a long while and is still used a lot. NetCDF has similar abilities and extended features. However, moving a field to use another file format is a challenge on its own. The converter that is being called in this script is written by a [Hidde Leijense](https://www.knmi.nl/research/publications?author=+Hidde+Leijnse) of KNMI. Hidde Leijense is often involved in Research with the Animal Movement Ecology of the University of Amsterdam as the KNMI and UvA collaborate frequently.