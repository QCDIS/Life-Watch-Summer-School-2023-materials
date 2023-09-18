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
3. Extra information
    1. [Aloft Data]
    2. [Open data repositories]
    3. 

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
# configuration
import os
param_radar = "" # denhelder,herwijnen,debilt
param_api_key = '' #
param_start_date = "" # %Y%m%d%H%M ; 202010152355
param_end_date = "" # %Y%m%d%H%M ; 202010152355
conf_storage_dir = f"{os.environ.get("USER")}/data" # Set the storage directory to your personal directory/data.
conf_convert_dir = 'convert'
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
#List-KNMI-files-Herwijnen-v10
## libraries
import requests
import pandas as pd

# The number of files this radar produces in a 24 hour period
daily_measurement_count_hrw = 288 
# Daily measurements interval in minutes = 24 Hours * 60 Minutes / 288 Daily measurement count 
measurement_interval_minutes = (24*60)/daily_measurement_count_hrw 
# Specific API url targetting the Herwijnen dataset
api_url_herwijnen = 'https://api.dataplatform.knmi.nl/open-data/v1/datasets/radar_volume_full_herwijnen/versions/1.0/files' 
# KNMI identifier that we need to construct a request
radar_code = "NL62" 
# configure 
api_url = api_url_herwijnen
## main
print(f'Searching for PVOL files for Herwijnen between {conf_target_start_day} and {conf_target_end_day} ')
start_date_timestamp = pd.Timestamp(conf_target_start_day)
end_date_timestamp = pd.Timestamp(conf_target_end_day)
date_range = pd.date_range(start_date_timestamp,end_date_timestamp)
# Considering we need a filename to search for files that occur AFTER that file - we pick the first date of the date range we construct previously
timestamp = date_range[0]
# Now, determine the number of files we expect for our request.
max_keys = len(date_range) * daily_measurement_count_hrw
# adjust timestamp to step back ONE measurement to include the requested timestamp
# Subtract 5 minutes from 00:00, this sets to previous day
timestamp = timestamp - pd.Timedelta(minutes=5)
# Format the timestamp such that the KNMI API understands
timestamp = timestamp.strftime("%Y%m%d%H%M")
# Construct the filename that is used to find files that were measured after the file.
start_after_filename_prefix = f"RAD_{radar_code}_VOL_NA_{timestamp}.h5"
# Request a response from the KNMI severs
list_files_response = requests.get(
                        f"{api_url}",
                        headers={"Authorization": param_api_key},
                        params={"maxKeys": max_keys, "startAfterFilename": start_after_filename_prefix},
                    )
list_files = list_files_response.json()
dataset_files = list_files.get("files")
# Rewrote this section to pull information from the dict (contained in the main list)
# we cant pass anything but primitives around between Cells, so we rewrite it into a nested list of strings
# note:
# idx0 = filename, idx1 = size, idx2 = lastModified
dataset_files = [list(dataset_file.values()) for dataset_file in dataset_files]
# Check the dates, the api sends X number of dates AFTER a timestamp. So, we need to ensure that we are not surpassing our end date. 
# Example: If one of our requested days != 288 files we will 'overshoot' our last requested day by (288*nDays)-(nActualFiles)).
filtered_dataset_files = []
for dataset_file in dataset_files:
    _,_,_,_,datetimestrext = dataset_file[0].split("_")
    datetimestr = datetimestrext.split(".")[0]
    file_timestamp = pd.Timestamp(datetimestr)
    if file_timestamp.strftime("%Y-%m-%d") in date_range:
        filtered_dataset_files.append(dataset_file)
print(f'Requested date range {date_range}')
print(f'Removed {len(dataset_files) - len(filtered_dataset_files)} entries from dataset files as they were outside the requested date range')
dataset_files = filtered_dataset_files
```
`Note:` The measurement interval for Dutch Meteorological Radars is 5 minutes. This means that each Radar can produce up to 288 measurements in a given day. Occasionally, radars fail to produce measurements and will therefore have less than 288 files.

Let's go ahead and request this code cell to be turned in a functional block, like we did with the previous exercise. Use the `component containeriser` to containerise the newly added code. Ensure to specify each of the types of variables correctly and to use the vol2bird base image. 

##### Exercise 3. Downloading Polar Volume data
Now we will be adding a block of code which can use the results from the previous block, files within our query, to download and store them at our convenience. Copy the code below and place it in your Jupyter notebook. Follow the same steps as previous exercises and containerise the code by using the `code containeriser` on your left.

```python
#Download-KNMI-files-Herwijnen-v10
##libraries
import requests
from pathlib import Path

knmi_paths = []
radar_code = 'NL/HRW'
api_url = "https://api.dataplatform.knmi.nl/open-data/v1/datasets/radar_volume_full_herwijnen/versions/1.0/files"
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
    p = Path(f"{conf_download_dir}/{year}/{month}/{day}/{filename}")
    knmi_paths.append('./{}'.format(str(p)))
    p.parent.mkdir(parents=True,exist_ok=True)
    p.write_bytes(dataset_file_response.content)
print(knmi_paths)
```
`Note:`

##### Exercise 4. Converting Polar Volume data
The files that we are downloading are HDF5 files which contain Polar Volumetric data. The data is currently structured according the the [data model of KNMI](https://www.knmi.nl/kennis-en-datacentrum/publicatie/knmi-hdf5-data-format-specification-v3-5). Most of our analysis methods expect a data model of [ODIM](https://www.eumetnet.eu/wp-content/uploads/2019/01/ODIM_H5_v23.pdf) which means we need to run a converter. The following code expects a list of files and will call a converter to convert the KNMI format to an ODIM format and returns us a list of files in ODIM format. 
```python
#Convert-KNMI-ODIM-v01-DUMMY
import subprocess
import pathlib
import h5py
import json
import sys
import shutil

class FileTranslatorFileTypeError(LookupError):
        '''raise this when there's a filetype mismatch derived from h5 file'''
    
def load_radar_db():
    """Load and return the radar database

    Output dict sample (wmo code is used as key):
    {
        11038: {'number': '1209', 'country': 'Austria', 'countryid': 'LOWM41', 'oldcountryid': 'OS41', 'wmocode': '11038', 'odimcode': 'atrau', 'location': 'Wien/Schwechat', 'status': '1', 'latitude': '48.074', 'longitude': '16.536', 'heightofstation': ' ', 'band': 'C', 'doppler': 'Y', 'polarization': 'D', 'maxrange': '224', 'startyear': '1978', 'heightantenna': '224', 'diametrantenna': ' ', 'beam': ' ', 'gain': ' ', 'frequency': '5.625', 'single_rrr': 'Y', 'composite_rrr': 'Y', 'wrwp': 'Y'},
        11052: {'number': '1210', 'country': 'Austria', 'countryid': 'LOWM43', 'oldcountryid': 'OS43', 'wmocode': '11052', 'odimcode': 'atfel', 'location': 'Salzburg/Feldkirchen', 'status': '1', 'latitude': '48.065', 'longitude': '13.062', 'heightofstation': ' ', 'band': 'C', 'doppler': 'Y', 'polarization': 'D', 'maxrange': '224', 'startyear': '1992', 'heightantenna': '581', 'diametrantenna': ' ', 'beam': ' ', 'gain': ' ', 'frequency': '5.6', 'single_rrr': 'Y', 'composite_rrr': ' ', 'wrwp': ' '},
        ...
    }
    """
    with open(
        './data/src/OPERA_RADARS_DB.json', mode="r"
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
    #output = proc.stderr.decode("utf-8")
    returncode = int(proc.returncode)
    return (out_fpath,returncode)
odim_paths = []
print(knmi_paths)
radar_db = load_radar_db()
for knmi_path in knmi_paths:   
    out_path_pvol_odim = knmi_path.replace('KNMI','ODIM')
    converter_results = knmi_to_odim(in_fpath = knmi_path,out_fpath = out_path_pvol_odim)
    print(converter_results)
    out_path_pvol_odim = converter_results[0]
    # remove the KNMI version
    pathlib.Path(knmi_path).unlink()
    # Determine name for our convention
    ibed_pvol_name = translate_knmi_filename(in_path_h5=out_path_pvol_odim)
    out_path_pvol_odim_tce = pathlib.Path(out_path_pvol_odim).parent.joinpath(ibed_pvol_name)
    shutil.move(src=out_path_pvol_odim,dst=out_path_pvol_odim_tce)
    odim_paths.append(out_path_pvol_odim_tce)

pvol_paths=odim_paths
```
`Note:` The code you copy executes a subprocess call to a custom C program written by [Hidde Leijense](https://www.knmi.nl/research/publications?author=+Hidde+Leijnse) of KNMI. Hidde Leijense is often involved in Research with the Animal Movement Ecology of the University of Amsterdam as the KNMI and UvA collaborate frequently.
We only convert the structure of data, not the file type. Both the KNMI and ODIM use the same filetype, [HDF5](https://www.hdfgroup.org/solutions/hdf5/). HDF5 is an industry standard hierarchial data storage. 

###### Exercise 5. Producing vertical profiles
We have all the blocks in place to search, download, convert Polar Volume (PVOL) data into a format that we can start analyzing but also process further into more derivatives, such as a vertical profile. A Vertical Profile (VP) is a way for us to 'summarize' the large amount of data that the Polar Volume shows us. A Polar Volume of The KNMI can have up to 16 different parameters measured per elevation angle (step). Each Radar measures a number of elevation (steps). For that elevation, it measures a host of different quantities and returns those as datasets per quantity and elevation angle. For us to interpret all those elevation angles and quantities is extremely difficult. 

To alleviate this difficulty, a Vertical Profile can be generated which analyses relevant quantities and elevation angles and attempts to determine the number of birds that have passed over an imaginary cross-section across the radar. It produces that information in a format which indicates per height bin a number of metrics. These metrics include the reflectivity in that specific bin, the expected number of birds / area and so on. The [github page of Vol2bird](https://github.com/adokter/vol2bird) contains more information on vol2bird and includes links to publications explaining how vol2bird works. 

We are now going to insert code that will also call a processor, vol2bird, to process PVOL data into VP data. 
```python
python code that calls vol2bird and makes vps
``` 
`Note:` There is a lot of Python code involved here. Most of this code is applying naming conventions and versioning. The AME group of the University of Amsterdam has over 5.5 million Vertical Profiles stored in two (synchronized) storage systems. In order to provide overview and structure a strong naming convention has been enforced within the AME group to great success.

###### Exercise 6. 
