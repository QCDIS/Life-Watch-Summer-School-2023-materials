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
    6. [Exercise 5. Producing vertical profiles](#exercise-5-producing-vertical-profiles)
    7. [Exercise 6. Creating the workflow](#exercise-6-creating-the-workflow)
    8. [Exercise 7. Producing data](#exercise-7-producing-data)
    9. [Exercise 8. Visualizing results](#exercise-8-visualizing-data)
3. [Open questions](#open-questions)
    1. [Fireworks and Birds](#fireworks-and-birds)
    2. [Peak migration events](#peak-migration-events-based-on-weather-variables)
4. [Publications and papers regarding migration](#publications-and-papers-regarding-migration)
5. [Open data repositories](#open-data-repositories)


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
First, we need to set certain parameters and configurations. We will be using a course specific API ([What is API?](https://www.ibm.com/topics/api)) key from the Dutch meteorological institute, KNMI (Koninklijk Nederlands Meteorologisch Instituut). With this key, we can authenticate ourselves within the data repository of [The KNMI Data Platform](https://dataplatform.knmi.nl/). From this platform, we will be retrieving Polar Volume Data which contains a host of parameters which can be used to detect biological echoes. 

`Important to know`: The API key used during this course is only valid for the duration of the course. KNMI has been kind and provided us with our own custom workshop API-key. They have done this to help us 'guarantee' to have access to the data without having to register or share the key with other anonymous users. As this is courtesy, please keep in mind to keep your queries short. Practically this means: Choose a day to retrieve data for, not a year. If you decide you want to see an effect over a large temporal time span, sample interval can be reduced. For example: every 15th day of the month, every X'th hour of the day. You do not have enough time during this course to download, convert and analyze large amounts of data nor will the API-key allow all students to download a large dataset. Rest assured, throughout the exercises you will be guided on how to query and stay within the `fair-use policy` and still get interesting results.  

```python
# configuration-v30
import os
param_radar = "" # denhelder,herwijnen
param_api_key = "" #
param_start_date = "" # %Y%m%dT%H:%M+TZ; 2019-12-31T23:00+00:00
param_end_date = "" # %Y%m%dT%H:%M+TZ; 2020-01-01T01:00+00:00
# radar configuration for the KNMI api
# datasetName, datasetVersion, api_url, radar code (odim)
conf_herwijnen = ['radar_volume_full_herwijnen',
                  1.0,
                  'https://api.dataplatform.knmi.nl/open-data/v1/datasets/radar_volume_full_herwijnen/versions/1.0/files',
                 'NL/HRW'] 
conf_denhelder = ['radar_volume_full_denhelder',
                 2.0,
                 'https://api.dataplatform.knmi.nl/open-data/v1/datasets/radar_volume_denhelder/versions/2.0/files',
                 'NL/DHL']
conf_radars = {'herwijnen' : conf_herwijnen,
              'denhelder' : conf_denhelder}
# Directories
conf_storage_dir = f"{os.environ.get('HOME')}/data" # Set the storage directory to your personal directory/data.
conf_knmi_dir = f'{conf_storage_dir}/KNMI' # where the KNMI files should be downloaded
conf_odim_dir = f'{conf_storage_dir}/ODIM' # converted from KNMI -> ODIM
conf_vp_dir = f'{conf_storage_dir}/vp' # processed in VP
conf_conf_dir = f'{conf_storage_dir}/conf' # configuration, such as radar DB
conf_clean_knmi_input = True # Remove input after processing KNMI format to ODIM format
check_dirs = [conf_storage_dir,
              conf_odim_dir,
              conf_vp_dir,
              conf_knmi_dir,
              conf_conf_dir]
for _dir in check_dirs:
    if not os.path.exists(_dir): # If this directory does not exist...
        os.makedirs(_dir) # Create the directory
# Reference files
conf_radar_db = f'{conf_conf_dir}/OPERA_RADARS_DB.json'
```

`Note:` There are two types of parameters recognized by NaaVRE Code analyzer: `param` and `conf`. The keyword `param` signifies that the user needs to supply this information when a workflow is executed. This could be a name of a Radar, a date, your password, etc. The keyword `conf` signifies that it holds information publicly shared. You would not enter your password here as it would be public to the world. You would use `conf` to indicate where your storage is located, what your maximum number of files needs to be, etc. 

On the left hand side, you can see the `component containeriser`. This allows us to store the code we just entered in the notebook as a code-block. We will eventually make a few more of these code-blocks and use them to make a workflow out of them. Click the `component containeriser` and containerise the code. Now your code cell is configured and you can go ahead and press `Create`. It will take a few minutes before it shows up in our cell catalogue. Instead of waiting, we are going to continue adding some more blocks.

But before we continue creating the next block, lets add an extra cell below and add some information such that we can use it to test our upcoming code blocks. You should have received an API key from one of the instructors. Create a new jupyter cell below the current one and and enter the information displayed below.  

```python
# DO NOT CONTAINERISE THIS CELL. DUMMY CELL FOR CONFIGURATION OF JUPYTER CODE ONLY.
param_radar = "denhelder" # denhelder,herwijnen
param_api_key = "" #
param_start_date = "2019-12-31T23:00+00:00" # %Y%m%dT%H:%M+TZ; 2019-12-31T23:00+00:00
param_end_date = "2020-01-01T01:00+00:00" # %Y%m%dT%H:%M+TZ; 2020-01-01T01:00+00:00
```

Now you could be wondering: But isn't `param` used to request input on execution of the workflow? Correct! What we need is to add a "dummy" configuration cell which will allow us to test our code that has been added in the jupyter cells. Once we know each individual cell works we are much more confident our workflow will execute.

Now we've added a configuration that targets the radar at Den Helder and requests Radar Data from 2019-12-31 23:00 -> 2021-01-01 01:00. This should give us data around New years eve. 

Lastly, `run` Cell 1 (# configuration-v30) and 2 (# DO NOT CONTAINERISE THIS CELL). Lastly, we need to add a reference database file which will help some of the functions determine the correct radar name to name the output files. Add `OPERA_RADARS_DB.json` to `/data/conf/`.

##### Exercise 2. Accessing KNMI Data Platform
The next step is to create a code-block which can use this information we've specified and communicate with the data repository of the KNMI. We are going to create a code-block which can ask the API which files it has for our search query. We will search by specifying three parameters: `start_date`, `end_date`, `radar`. The API will then return us (if successful) a list of file names corresponding to Radar Measurements for our query. Copy the code below into a new cell in Jupyter.

```python
# list-knmi-files-v30
# Notes:
# Timestamps in iso8601
# 2020-01-01T00:00+00:00
# Libraries
import requests
# configure 
start_ts = param_start_date
end_ts = param_end_date
datasetName, datasetVersion, api_url, _ = conf_radars.get(param_radar)
params = {'datasetName' : datasetName,
          'datasetVersion' : datasetVersion,
          'maxKeys' : 10,
          'sorting' : "asc",
          'orderBy' : "created",
          'begin' : start_ts,
          'end' : end_ts,
         }
# Request a response from the KNMI severs
# Try the next page tokens
dataset_files = []
while True:
    list_files_response = requests.get(url = api_url,
                            headers={"Authorization": param_api_key},
                            params = params)
    list_files = list_files_response.json()
    dset_files = list_files.get("files")
    dset_files = [list(dset_file.values()) for dset_file in dset_files]
    dataset_files += dset_files
    nextPageToken = list_files.get("nextPageToken")
    if not nextPageToken:
        break
    else:
        params.update({'nextPageToken' : nextPageToken})
# Rewrote this section to pull information from the dict (contained in the main list)
# we cant pass anything but primitives around between Cells, so we rewrite it into a nested list of strings
# note:
# idx0 = filename, idx1 = size, idx2 = lastModified, idx3 = created
# Subset to a different interval

# KNMI outputs per 5 minutes, per 15 is less of a heavy hit on downloads and processing
# Quick and dirty way to only keep the 15 minute measurements
filtered_list = []
for dataset_file in dataset_files:
    minute = int(dataset_file[0].split('_')[-1].split('.')[0][-2:])
    if minute in [0,15,30,45]:
        filtered_list.append(dataset_file)
        
dataset_files = filtered_list
print(f"Found {len(dataset_files)} files")
print(dataset_files)
```

`Note:` The measurement interval for Dutch Meteorological Radars is 5 minutes. This means that each Radar can produce up to 288 measurements in a given day. Occasionally, radars fail to produce measurements and will therefore have less than 288 files. This code block has been adjusted such that it yields files in an interval of 15 minutes as opposed to 5 minutes. This is deliberately done in an attempt to reduce the strain on the servers and services. You are allowed to set it back to 5 minutes but this is not advisable for the duration of the course. 

Test the code by executing the cell. You should see a print out that 8 files have been found. 

Let's go ahead and request this code cell to be turned in a functional block, like we did with the previous exercise. Use the `component containeriser` to containerise the newly added code. Ensure to specify each of the types of variables correctly and to use the vol2bird base image. 

##### Exercise 3. Downloading Polar Volume data
Now we will be adding a block of code which can use the results from the previous block, files within our query, to download and store them at our convenience. Copy the code below and place it in your Jupyter notebook. Follow the same steps as previous exercises. Test the code by executing the cell in Jupyter and containerise the code by using the `code containeriser` on your left.

```python
#Download-KNMI-v30
##libraries
import requests
from pathlib import Path
_, _, api_url, radar_code = conf_radars.get(param_radar)
knmi_pvol_paths = []
n_files = len(dataset_files)
print(f"Starting download of {n_files} files.")
idx = 1
for dataset_file in dataset_files:
    print(f"Downloading file {idx}/{n_files}")
    filename = dataset_file[0]
    endpoint = f"{api_url}/{filename}/url"
    get_file_response = requests.get(endpoint, headers={"Authorization": param_api_key})
    download_url = get_file_response.json().get("temporaryDownloadUrl")
    dataset_file_response = requests.get(download_url)
    fname_parts = filename.split('_')
    fname_date_part = fname_parts[-1].split('.')[0]
    year = fname_date_part[0:4]
    month = fname_date_part[4:6]
    day = fname_date_part[6:8]
    p = Path(f"{conf_knmi_dir}/{year}/{month}/{day}/{filename}")
    knmi_pvol_paths.append('{}'.format(str(p)))
    p.parent.mkdir(parents=True,exist_ok=True)
    p.write_bytes(dataset_file_response.content)
    idx += 1
print(knmi_pvol_paths)
```

##### Exercise 4. Converting Polar Volume data
The files that we are downloading are HDF5 files which contain Polar Volumetric data. The data is currently structured according the the [data model of KNMI](https://www.knmi.nl/kennis-en-datacentrum/publicatie/knmi-hdf5-data-format-specification-v3-5). Most of our analysis methods expect a data model of [ODIM](https://www.eumetnet.eu/wp-content/uploads/2019/01/ODIM_H5_v23.pdf) which means we need to run a converter. The following code expects a list of files and will call a converter to convert the KNMI format to an ODIM format and returns us a list of files in ODIM format. Copy the code below into a new Jupyter cell and test it. If the code works containerise it using the `code containeriser`

```python
#KNMI-to-ODIM-converter-v30
import subprocess
import pathlib
import h5py
import json
import sys
import shutil

class FileTranslatorFileTypeError(LookupError):
        '''raise this when there's a filetype mismatch derived from h5 file'''  
def load_radar_db(radar_db_path):
    """Load and return the radar database

    Output dict sample (wmo code is used as key):
    {
        11038: {'number': '1209', 'country': 'Austria', 'countryid': 'LOWM41', 'oldcountryid': 'OS41', 'wmocode': '11038', 'odimcode': 'atrau', 'location': 'Wien/Schwechat', 'status': '1', 'latitude': '48.074', 'longitude': '16.536', 'heightofstation': ' ', 'band': 'C', 'doppler': 'Y', 'polarization': 'D', 'maxrange': '224', 'startyear': '1978', 'heightantenna': '224', 'diametrantenna': ' ', 'beam': ' ', 'gain': ' ', 'frequency': '5.625', 'single_rrr': 'Y', 'composite_rrr': 'Y', 'wrwp': 'Y'},
        11052: {'number': '1210', 'country': 'Austria', 'countryid': 'LOWM43', 'oldcountryid': 'OS43', 'wmocode': '11052', 'odimcode': 'atfel', 'location': 'Salzburg/Feldkirchen', 'status': '1', 'latitude': '48.065', 'longitude': '13.062', 'heightofstation': ' ', 'band': 'C', 'doppler': 'Y', 'polarization': 'D', 'maxrange': '224', 'startyear': '1992', 'heightantenna': '581', 'diametrantenna': ' ', 'beam': ' ', 'gain': ' ', 'frequency': '5.6', 'single_rrr': 'Y', 'composite_rrr': ' ', 'wrwp': ' '},
        ...
    }
    """
    with open(
        radar_db_path, mode="r"
    ) as f:
        radar_db_json = json.load(f)
    radar_db = {}
    # Reorder list to a usable dict with sub dicts which we can search with wmo codes
    for radar_dict in radar_db_json:
        try:
            wmo_code = int(radar_dict.get("wmocode"))
            radar_db.update({wmo_code: radar_dict})
        except Exception:  # Happens when there is for ex. no wmo code.
            pass
    return radar_db
def translate_wmo_odim(radar_db,wmo_code):
    """
    """
    if not isinstance(wmo_code,int):
        raise ValueError("Expecting a wmo_code [int]")
    else:
        pass
    odim_code = radar_db.get(wmo_code).get("odimcode").upper().strip() # Apparently, people sometimes forget to remove whitespace..
    return odim_code
def extract_wmo_code(in_path):
    with h5py.File(in_path, mode="r") as f:
        #DWD Specific
        #Main attributes
        what = f['what'].attrs
        #Source block
        source = what.get('source')
        source = source.decode("utf-8")
        # Determine if we are dealing with a WMO code or with an ODIM code set
        # Example from Germany where source block is set as WMO
        # what/source: "WMO:10103"
        # Example from The Netherlands where source block is set as a combination of ODIM and various codes
        # what/source: RAD:NL52,NOD:nlhrw,PLC:Herwijnen
        source_list = source.split(sep=",")
    wmo_code = [string for string in source_list if "WMO" in string]
    # Determine if we had exactly one WMO hit
    if len(wmo_code) == 1:
        wmo_code = wmo_code[0]
        wmo_code = wmo_code.replace("WMO:","")
    # No wmo code found, most likeley dealing with a dutch radar
    elif len(wmo_code) == 0:
        rad_str = [string for string in source_list if "RAD" in string]

        if len(rad_str) == 1:
                rad_str = rad_str[0]
        else:
            print("Something went wrong with determining the rad_str and it wasnt WMO either, exitting")
            sys.exit(1)
        # Split the rad_str
        rad_str_split = rad_str.split(":")
        # [0] = RAD, [1] = rad code
        rad_code = rad_str_split[1]

        rad_codes = {"NL52" : "6356",
                     "NL51" : "6234",
                     "NL50" : "6260"}

        wmo_code = rad_codes.get(rad_code)
    return int(wmo_code)
def translate_knmi_filename(in_path_h5):
    wmo_code = extract_wmo_code(in_path_h5)
    odim_code = translate_wmo_odim(radar_db,wmo_code)
    with h5py.File(in_path_h5, mode = "r") as f:
        what = f['what'].attrs
        #Date block
        date = what.get('date')
        date = date.decode("utf-8")
        #Time block
        time = what.get('time')
        #time = f['dataset1/what'].attrs['endtime']
        time = time.decode("utf-8")
        hh = time[:2]
        mm = time[2:4]
        ss = time[4:]
        time = time[:-2] # Do not include seconds
        #File type
        filetype = what.get('object')
        filetype = filetype.decode("utf-8")
        if filetype != "PVOL":
            raise FileTranslatorFileTypeError("File type was NOT pvol")
    name = [odim_code,filetype.lower(),date + "T" + time,str(wmo_code) + ".h5"]
    ibed_fname = "_".join(name)
    return ibed_fname
def knmi_to_odim(in_fpath,out_fpath):
    """
    Converter usage:
    Usage: KNMI_vol_h5_to_ODIM_h5 ODIM_file.h5 KNMI_input_file.h5

    Returns out_fpath and returncode
    """
    converter = 'KNMI_vol_h5_to_ODIM_h5'
    command = [converter,
               out_fpath,
               in_fpath]         
    proc = subprocess.run(command,stderr=subprocess.PIPE)
    output = proc.stderr.decode("utf-8")
    returncode = int(proc.returncode)
    return (out_fpath,returncode)
odim_pvol_paths = []
radar_db = load_radar_db(conf_radar_db)
for knmi_path in knmi_pvol_paths:
    out_path_pvol_odim = pathlib.Path(knmi_path.replace('KNMI','ODIM'))
    if not out_path_pvol_odim.parent.exists():
        out_path_pvol_odim.parent.mkdir(parents=True,exist_ok=False)
    converter_results = knmi_to_odim(in_fpath = knmi_path,out_fpath = out_path_pvol_odim)
    # remove the KNMI version
    if conf_clean_knmi_input:
        pathlib.Path(knmi_path).unlink()
        if not any(pathlib.Path(knmi_path).parent.iterdir()):
                   pathlib.Path(knmi_path).parent.rmdir()
    # Determine name for our convention
    ibed_pvol_name = translate_knmi_filename(in_path_h5=out_path_pvol_odim)
    out_path_pvol_odim_tce = pathlib.Path(out_path_pvol_odim).parent.joinpath(ibed_pvol_name)
    shutil.move(src=out_path_pvol_odim,dst=out_path_pvol_odim_tce)
    odim_pvol_paths.append(out_path_pvol_odim_tce)
print(odim_pvol_paths)
```

`Note:` The code you copy executes a subprocess call to a custom C program written by [Hidde Leijense](https://www.knmi.nl/research/publications?author=+Hidde+Leijnse) of KNMI. Hidde Leijense is often involved in Research with the Animal Movement Ecology of the University of Amsterdam as the KNMI and UvA collaborate frequently.
We only convert the structure of data, not the file type. Both the KNMI and ODIM use the same filetype, [HDF5](https://www.hdfgroup.org/solutions/hdf5/). HDF5 is an industry standard hierarchial data storage. 

###### Exercise 5. Producing vertical profiles
We have all the blocks in place to search, download, convert Polar Volume (PVOL) data into a format that we can start analyzing but also process further into more derivatives, such as a vertical profile. A Vertical Profile (VP) is a way for us to 'summarize' the large amount of data that the Polar Volume shows us. A Polar Volume of The KNMI can have up to 16 different parameters measured per elevation angle (step). Each Radar measures a number of elevation (steps). For that elevation, it measures a host of different quantities and returns those as datasets per quantity and elevation angle. For us to interpret all those elevation angles and quantities is extremely difficult. 

To alleviate this difficulty, a Vertical Profile can be generated which analyses relevant quantities and elevation angles and attempts to determine the number of birds that have passed over an imaginary cross-section across the radar. It produces that information in a format which indicates per height bin a number of metrics. These metrics include the reflectivity in that specific bin, the expected number of birds / area and so on. The [github page of Vol2bird](https://github.com/adokter/vol2bird) contains more information on vol2bird and includes links to publications explaining how vol2bird works. 

We are now going to insert code that will also call a processor, vol2bird, to process PVOL data into VP data. Copy the code below in a new Jupyter cell. Test that the code works and if it does containerise it using the `code containeriser`. 

```python
##PVOL-VP-converter-v30

import pandas as pd
import re
def load_radar_db(radar_db_path):
    """Load and return the radar database
    Output dict sample (wmo code is used as key):
    {
        11038: {'number': '1209', 'country': 'Austria', 'countryid': 'LOWM41', 'oldcountryid': 'OS41', 'wmocode': '11038', 'odimcode': 'atrau', 'location': 'Wien/Schwechat', 'status': '1', 'latitude': '48.074', 'longitude': '16.536', 'heightofstation': ' ', 'band': 'C', 'doppler': 'Y', 'polarization': 'D', 'maxrange': '224', 'startyear': '1978', 'heightantenna': '224', 'diametrantenna': ' ', 'beam': ' ', 'gain': ' ', 'frequency': '5.625', 'single_rrr': 'Y', 'composite_rrr': 'Y', 'wrwp': 'Y'},
        11052: {'number': '1210', 'country': 'Austria', 'countryid': 'LOWM43', 'oldcountryid': 'OS43', 'wmocode': '11052', 'odimcode': 'atfel', 'location': 'Salzburg/Feldkirchen', 'status': '1', 'latitude': '48.065', 'longitude': '13.062', 'heightofstation': ' ', 'band': 'C', 'doppler': 'Y', 'polarization': 'D', 'maxrange': '224', 'startyear': '1992', 'heightantenna': '581', 'diametrantenna': ' ', 'beam': ' ', 'gain': ' ', 'frequency': '5.6', 'single_rrr': 'Y', 'composite_rrr': ' ', 'wrwp': ' '},
        ...
    }
    """
    with open(
        radar_db_path, mode="r"
    ) as f:
        radar_db_json = json.load(f)
    radar_db = {}
    # Reorder list to a usable dict with sub dicts which we can search with wmo codes
    for radar_dict in radar_db_json:
        try:
            wmo_code = int(radar_dict.get("wmocode"))
            radar_db.update({wmo_code: radar_dict})
        except Exception:  # Happens when there is for ex. no wmo code.
            pass
    return radar_db
def translate_wmo_odim(radar_db, wmo_code):
    """"""
    # class FileTranslatorFileTypeError(LookupError):
    #    '''raise this when there's a filetype mismatch derived from h5 file'''
    if not isinstance(wmo_code, int):
        raise ValueError("Expecting a wmo_code [int]")
    else:
        pass
    odim_code = (
        radar_db.get(wmo_code).get("odimcode").upper().strip()
    )  # Apparently, people sometimes forget to remove whitespace..
    return odim_code
def extract_wmo_code(in_path):
    with h5py.File(in_path, "r") as f:
        # DWD Specific
        # Main attributes
        what = f["what"].attrs
        # Source block
        source = what.get("source")
        source = source.decode("utf-8")
        # Determine if we are dealing with a WMO code or with an ODIM code set
        # Example from Germany where source block is set as WMO
        # what/source: "WMO:10103"
        # Example from The Netherlands where source block is set as a combination of ODIM and various codes
        # what/source: RAD:NL52,NOD:nlhrw,PLC:Herwijnen
        source_list = source.split(sep=",")
    wmo_code = [string for string in source_list if "WMO" in string]
    # Determine if we had exactly one WMO hit
    if len(wmo_code) == 1:
        wmo_code = wmo_code[0]
        wmo_code = wmo_code.replace("WMO:", "")
    # No wmo code found, most likeley dealing with a dutch radar
    elif len(wmo_code) == 0:
        rad_str = [string for string in source_list if "RAD" in string]
        if len(rad_str) == 1:
            rad_str = rad_str[0]
        else:
            print(
                "Something went wrong with determining the rad_str and it wasnt WMO either, exiting"
            )
            sys.exit(1)
        # Split the rad_str
        rad_str_split = rad_str.split(":")
        # [0] = RAD, [1] = rad code
        rad_code = rad_str_split[1]
        rad_codes = {"NL52": "6356", "NL51": "6234", "NL50": "6260"}
        wmo_code = rad_codes.get(rad_code)
    return int(wmo_code)
def vol2bird(in_file, out_dir, radar_db, add_version=True, add_sector=False):
    # Construct output file
    date_regex = "([0-9]{8})"
    if add_version == True:
        version = "v0-3-20"
        suffix = pathlib.Path(in_file).suffix
        in_file_name = pathlib.Path(in_file).name
        in_file_stem = pathlib.Path(in_file_name).stem
        #
        out_file_name = in_file_stem.replace("pvol", "vp")
        out_file_name = "_".join([out_file_name, version]) + suffix
        # odim = odim_code(out_file_name)
        wmo = extract_wmo_code(in_file)
        odim = translate_wmo_odim(radar_db, wmo)
        datetime = pd.to_datetime(re.search(date_regex, out_file_name)[0])
        ibed_path = "/".join(
            [
                odim[:2],
                odim[2:],
                str(datetime.year),
                str(datetime.month).zfill(2),
                str(datetime.day).zfill(2),
            ]
        )
        # check if we need to make this dir
        out_file = "/".join([out_dir, ibed_path, out_file_name])
        out_file_dir = pathlib.Path(out_file).parent
        if not out_file_dir.exists():
            out_file_dir.mkdir(parents=True)
    command = ["vol2bird", in_file, out_file]
    result = subprocess.run(command, stderr=subprocess.DEVNULL)
    return [in_file, out_file]
vertical_profile_paths = []
radar_db = load_radar_db(conf_radar_db)
for odim_pvol_path in odim_pvol_paths:
    output_file = vol2bird(odim_pvol_path, conf_vp_dir, radar_db)
    vertical_profile_paths.append(output_file)
print(vertical_profile_paths)
``` 

`Note:` There is a lot of Python code involved here. Most of this code is applying naming conventions and versioning. The AME group of the University of Amsterdam has over 5.5 million Vertical Profiles stored in two (synchronized) storage systems. In order to provide overview and structure a strong naming convention has been enforced within the AME group to great success.

###### Exercise 6. Creating the workflow
Now we have a few functional blocks that we can start to connect. Click the '+' on the top to create a new tab. This tab should have an option called `experiment manager`. Click the `experiment manager` and open the cell catalogue on the right (click `+`). From the cell catalogue pick the following cells:

```
1. List-KNMI-files-v30
2. Download-KNMI-files-v30
3. Converter-KNMI-files-v30
4. PVOL-VP-processor-v30
```

These blocks will now be made available to you in the `experiment manager`. We can now start to connect the individual blocks and to allow the NaaVRE environment to transfer the information between each block and handle its execution. In the previous exercises we had to specify which input and output variables each block had. Now, we can draw lines between all input and output of each block. 

Connect all blocks in a fashion such that the following workflow is created: 

```
List-KNMI-files-v30 -> Download-KNMI-files-v30 -> Converter-KNMI-files-v30 -> PVOL-VP-processor-v30
```

###### Exercise 7. Producing data
We now have a functional workflow which can retrieve and process any Radar data from the Herwijenen or Den Helder Meteorological Radars. We will now configure our workflow in such a manner that we will zoom in on a known mass migration event. The mass migration event was logged by [Judy Shamoun-Baranes](https://www.uva.nl/profiel/s/h/j.z.shamoun-baranes/j.z.shamoun-baranes.html) head of Animal Movement Ecology, University of Amsterdam.

Now, once we submit the workflow we will be prompted for the following information: Start date, end date, radar and API key. Fill in the following information:

`radar` = `denhelder`

`start_date` = `2022-10-10T18:00+00:00`

`end_date` `2022-10-11T06:00+00:00`

`api_key` = `yourapikey`

Execute your workflow by by pressing the `+` button and click the `Execute workflow` button. 

###### Exercise 8. Visualizing data
The most convenient way to visualize radar data is by using the library bioRad, written in R. We can use the Jupyter notebook to create an R session which has bioRad installed. Click the `+` button on the top right and start a new R notebook. Paste the code in the R notebook cell and run it. 

```R
# Load bioRad
library('bioRad')
# Specify the location of where Vertical Profiles are stored
vp_dir <- "/home/joyvan/data/vp/"
# Return the file paths
vp_file_paths <- list.files(vp_dir, full.name = TRUE, pattern = "*.h5", recursive = TRUE)
# Read the file paths and generate a list of Vertical Profile Objects
vp_list <- bioRad:::read_vpfiles(vp_file_paths)
# convert the list of vertical profiles into a time series:
vpts <- bind_into_vpts(vp_list)
# regularize VPTS
reg_vpts <- regularize_vpts(vpts)
plot(reg_vpts)
```

The code generates a Vertical Profile Time Series which indicates per height bin a number of birds / km3. The library bioRad is usefull library to analyze and interact with Radar Data and derivatives. The data you've produced can be analyzed with the bioRad package. [RadAero2022](https://adriaandokter.com/bioRad/articles/rad_aero_22.html) are introductory exercises on bioRad where you will be reading Polar Volumes, Vertical Profiles generating and Range Bias Corrected data. All data generated in this course can be used in the RadAero2022 exercises in biRad. You can skip downloading vertical profiles and polar volumes in the RadAero2022 exercises as you have just generated those in your RAVRE environment. It is recommended to have a look and try some of these commands on the data you've just produced. Try creating a Plan Position Indicator from one of the Polar Volumes you've generated. 

# Open questions
## Fireworks and Birds. 
Quite some research is being done to determine the effect and scope of fireworks on birds. I think we can all imagine the effect fireworks has on birds - but can we see this effect in Radar Data?

Objective: Generate Polar Volumes and Vertical Profiles for either (or both) radars around New Years Eve. 

First, visualize and interpret one New Years Eve. Once done - create a larger temporal span (i.e. include more years) and interpret the effect of fireworks on birds over the year. 
* Is the effect of Fireworks increasing, decreasing, other?
* What happened during the recent Corona Pandemic. Did we see an increase, decrease, other effect? 
  1. During the Corona Pandemic, The Netherlands banned the use of all fireworks by it's citizens. 
  2. During 'normal' years The Netherlands allows its citizens to be launch a great variation of fireworks. 

### Extra information
You can e-mail your findings to `b.c.wijers@uva.nl` with the title `Lifewatch Summerschool 2023 RAVRE - Fireworks`. We will then discuss your findings with the Animal Movement Ecology group and provide feedback. 

## Peak migration events based on Weather variables
As mentioned in the introduction, it seems that birds tend to wait for favourable weather conditions before departing. Sometimes it takes too long to find these weather conditions and the birds depart in seemingly bad conditions. Therefore, we can't fully say when they depart but we can have a good estimate of when could be a good moment. By reviewing weather variables we can therefore estimate when birds could be departing en masse. 

Objective: Use weather variables to estimate when there would be a peak migration event. Once identified, generate PVOL and VP and visualize the results. 

### Information
The website you can use to view a host of weather variables can be found here: [earth.nullschool.net](https://earth.nullschool.net/#2017/07/26/2200Z/wind/surface/level/orthographic=7.88,51.69,2762/loc=7.511,49.824). For Autumn migration, winds originating from the North and heading South, SouthWest seem to be favourable for passerine migration. In Spring (February - May) winds originating from the South and heading North NorthEast seem to be favourable. Furthermore, regardless of season rain is not favourable. 

To get you started, a few dates will be shown to you of where there is a peak migration event positive weather variables. Try some of the dates and see if you can see the favourable wind conditions. Also see what the wind direction was before the 8th of october and after the 13th of october. After that, use the earth.nullschool.net website to find more moments in time that could warrant peak migration. Pick one and visualize and verify.

Night of 8th of October, 2022
```python
start_date = '2022-10-08T18:00:00+00:00'
end_date = '2022-08-09T06:00:00+00:00'
radar = 'herwijnen' # denhelder not available
```
Night of 9th of October, 2022
```python
start_date = '2022-10-09T18:00:00+00:00'
end_date = '2022-08-10T06:00:00+00:00'
radar = 'herwijnen' # denhelder not available
```
Night of 10th of October, 2022
```python
start_date = '2022-10-10T18:00:00+00:00'
end_date = '2022-08-11T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
Night of 11th of October, 2022
```python
start_date = '2022-10-11T18:00:00+00:00'
end_date = '2022-08-12T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
Night of 12th of October, 2022
```python
start_date = '2022-10-12T18:00:00+00:00'
end_date = '2022-08-13T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
This night specifically is rather spectacular and has very telling weather variables before and during.
Night of 18th of Ocotber, 2022
```python
start_date = '2021-10-18T18:00:00+00:00'
end_date = '2022-08-19T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
The following dates are peak migration nights but the weather variables are not favourable. Have a look at these dates and also look at the weather variables before and after the migration. We expect the birds to have left as they waited long enough and decided to migrate in less optimal conditions. 
Night of 6th of Ocotber, 2021
```python
start_date = '2021-10-06T18:00:00+00:00'
end_date = '2022-08-07T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
Night of 7th of Ocotber, 2021
```python
start_date = '2021-10-07T18:00:00+00:00'
end_date = '2022-08-08T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
Night of 8th of Ocotber, 2021
```python
start_date = '2021-10-08T18:00:00+00:00'
end_date = '2022-08-09T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
Night of 12th of Ocotber, 2021
```python
start_date = '2021-10-06T18:00:00+00:00'
end_date = '2022-08-07T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```
Night of 13th of Ocotber, 2021
```python
start_date = '2021-10-06T18:00:00+00:00'
end_date = '2022-08-07T06:00:00+00:00'
radar = 'herwijnen' # or denhelder
```

### Extra information
You can e-mail your findings to `b.c.wijers@uva.nl` with the title `Lifewatch Summerschool 2023 RAVRE - Peak migration`. We will then discuss your findings with the Animal Movement Ecology group and provide feedback. 

# Publications and papers regarding migration
* Kranstauber B, Bouten W, van Gasteren H, Shamoun-Baranes J. 2022. Ensemble
predictions are essential for accurate bird migration forecasts for conservation
and flight safety. Ecological Solutions and Evidence 3(3)
* van Dobben H, 1953. Bird migration in the Netherlands. Ibis (95)
* van Doren BM, Horton KG. 2018. A continental system for forecasting bird migration.
Science 361:1115–1118.
* Richardson WJ. 1978. Timing and Amount of Bird Migration in Relation to Weather:
A Review. Oikos 30:224–272.

# Open data repositories
* Vertical profiles in Europe
    * [aloftdata](https://aloftdata.eu/)
* Polar Volume data The Netherlands
    * [knmi](https://data.knmi.nl)
* Polar Volume data Germany
    * [dwd](https://opendata.dwd.de/)
* Polar Volume Data Danmark
    * [dmi](https://confluence.govcloud.dk/display/FDAPI)
