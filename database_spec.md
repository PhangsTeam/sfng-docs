### Data Store Specifications for the Star Formation in Nearby Galaxies Collaboration

#### Objectives

1. Provide a single-point repository that serves science-ready data products to the collaboration.
2. Establish a stable data structure that can be algorthmically traversed to build up indices of file and their associated metadata.
3. The store should be flexible enough to include both telescope products and downstream analysis outputs (e.g., SFR maps)
4. The storage should be human-navigable and use naming conventions that are easy to interpret.
5. The method should provide a method for release of data to the general community.

Here are a few use cases that the database structure should be able to solve.

* You want to find all CO(1-0) maps

#### Structure

The data storage will be a single hierarchical file store.  Only certain file types should be stored in the directory tree.

    ./
    ./SFNG-INDEX.fits
    ./data/
    ./derived/
    ./docs/
    ./release/
    ./tables/
    ./uncalibrated/

The intent of the directories is as follows:

* `./data/` -- Directories containing data files corresponding to a single, unified observational data set.  Contents are calibrated images in sky brightness units (e.g., `K`, `MJY/SR`, `JY/PIX`, `W/M**2`)
* `./derived/` -- Derived science products from the data in `./data/`.  Examples would be star formation rates, gas surface density maps.
* `./docs/` -- Annoying documents like this one. 
* `./release/` -- Bundles of files for particular community releases.
* `./tables/` -- Tabular data including catalogs, line-of-sight sample databases.
* `./uncalibrated/` -- Here there be dragons.

The `SFNG-INDEX.fits` is a procedurally generated FITS BINTABLE index of the `./data/` and `./derived/` trees.  Downloading this file should allow queries of what files are available in the whole structure.

There is a single coarse organizational scheme to facilitate human navigation consisting of one layer of subdirectories that will enable rapid traversal to specific types of data.

    ./data/FIR/
    ./data/NIR/
    ./data/Optical/
    ./data/Radio/
    ./data/Submm/
    ./data/UV/
    
Some tracers, in particular are called out based on their particular utility for this line of work.

    ./data/CO/
    ./data/HI/
    ./data/Ha/

#### The `./data/` directories.

Each subdirectory in the `./data/` tree represents a unified observational campaign of one or more galaxies.  A directory can contain multiple tracers such as the different spectral windows in an ALMA map or mutiple optical bands.  Examples of current data that would get their own directory: HERACLES, KINGFISH, THINGS, PAWS.  Data from different tracers in the same waveband (e.g., all the SPIRE data from KINGFISH) should be included in the same directory.

##### Minimum file specifications for `./data/`

Data stored in the `./data/` tree are fully calibrated spectroscopy, images, and datacubes.  These should be science ready and carry with them a minimum of metadata to make them useful in the subsequent analysis.  Files will be included in the `./data/` tree if they fulfill all of the following specifications. 

1. Data should be calibrated to the common standard of closest to sky units as possible without requiring assumptions (e.g., source beam coupling).
2. FITS files are strongly preferred.
3. For images and data cubes, minimum valid WCS compliance that is readable in IDL via Astronomy library and in Python via astropy.  Of note, Stokes axes should be handled carefully and not result in singular WCS matrices (I'm looking at you, PdBI).

_Desiderata_

4. Specified units using the `BUNIT` keyword. 
5. Specified resolution using `BMAJ`, `BMIN` and `BPA`.  Standard is to have `BMIN` and `BMAX` are in decimal degrees and `BPA` is in degrees east of north.  
6. For spectral line data, the rest frequency in units specified by `RESTFRQ` in units of Hz.
7. For spectral line data, the spectral resolution in units of the spectral axis using keyword `SPECRES`
8. Individual spectra should be grouped by galaxies and stored as FITS BINTABLES.  
9. Units for the axes using `CUNITn` keywords.  This is quite rare in FITS files.

Missing keywords that describe the data, especially `BUNIT`, `BMAJ`, `BMIN`, `BPA`, `SPECRES`,`CUNITn` which are common for radio data but less so for optical, can be specified at the beginning of the README file for all files in the directory.

These files represent sky quantities and have *not* been processed for specific scientific outcomes (e.g., diffuse 24 micron emission correction).

##### Naming Conventions 

Individual files should represent individual galaxies or continouous maps of groups (e.g., NGC 5194/5195).  Each file will be stored with their names given as the NED Preferred name for the object as the leading value.  Spaces are replaced with underscores: `_` and separated from the remainder of the file name by a `.`.  Consider the FITS data from the THINGS survey [`NGC_4826_NA_CUBE_THINGS.FITS`](http://www.mpia.de/THINGS/Data_files/NGC_4826_NA_CUBE_THINGS.FITS) should be stored as `MESSIER_064.NGC_4826_NA_CUBE_THINGS.FITS`.  Cleaving to this naming convention will help us traverse our data both by waveband and by galaxy.

The NED canonical name can be established directly by query NED by hand, using `astroquery` in python 

```
   from astroquery.ned import Ned
   (Ned.query_object('NGC 598'))['Object Name'].data.data[0]
```

Or the direct URL search, which returns parsable XML

    http://ned.ipac.caltech.edu/cgi-bin/nph-objsearch?%20extend=no&of=xml_main&objname=NGC+598
    
The preferred name is the first `TD` entry in the returned XML.  Multiple objects in the same file could be handled by soft links.

_Alternatively, we can just re-edit all the fits files to include the NED canonical name in the `OBJECT` keyword._

##### Survey metadata information

Each directory in the `./data/` hierarchy will have a `SurveyName_README.txt` file (e.g., `THINGS_README.txt` where `THINGS` is also the directory name), which briefly explains where the data are from (URLs are great), what they represent, plus caveats and warnings as necessary.  If certain metadata are missing from the FITS files but appropriate for all files in the directory, they should be specified at the beginning of the README file and separated from the rest of the file by a single line containing three hyphens. Each line needs to be parsable as the structure `KEYWORD = VALUE` where the `KEYWORD` is the FITS keyword, the `VALUE` is read as a string and the separator is ` = `.  String `VALUES` representing numbers should cast to their appropriate types in IDL and Python.  Metadata specified in the README will be superseded by metadata in the actual files.  For example, `THINGS_README.txt` might have the structure:

    BUNIT = K
    BMAJ = 4.1667e-3
    BMIN = 4.1667e-3
    BPA = 0.0
    CUNIT3 = Hz
    ---
    The HI Nearby Galaxy survey data by Walter et al. (2008), AJ, 136, 2563.  VLA survey of nearby galaxies in 21-cm line emission.
    URL: http://www.mpia.de/THINGS/Data.html
    
There are the opportunities to add other keywords here.

##### Uncertainty information (suggestion)

Since summary products of the data and understanding the quality of the data in each file requires understand the noise level, it would be ideal to add keyword information to the headers that captures the uncertainty in each file.  This would probably mean making up a FITS keyword (`REPUNC` for representative uncertainty), or adding a `HISTORY` card. Alternatively, this could be described in keyword-value basis in the README file.

#### Derived data products

The `derived` hierarchy should contain the information that can be deduced from the sky brightness images in the `data` subject to physical models or data processing.  Considering spectral line data cubes, the data cube would be in `data` and the (masked) moment maps would sit in `derived`.  Data in `derived` have the same minimum FITS standards as in the `data` directory, but likely would have much more information specified in their respective `README` files.
