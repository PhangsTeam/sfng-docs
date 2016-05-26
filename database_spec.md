### Data Store Specifications for the Star Formation in Nearby Galaxies Collaboration

This document outlines our data specification for the SFNG collaboration.  The data store fulfills several goals.

#### Objectives

1. Provide a single-point repository for science-ready data products to the collaboration.
2. Establish a stable data structure that can be algorthmically traversed to build up derived products.
3. The store should be flexible enough to include both telescope products and downstream analysis outputs (e.g., SFR maps)
4. The storage should be human-navigable and use naming conventions that are easy to interpret.
3. Provide a method for release of data to the general community.

#### Structure

The data storage will be a single hierarchical file store.  Only certain file types should be stored in the directory tree.

    ./
    ./SFNG-INDEX.fits
    ./data/
    ./derived/
    ./docs/
    ./tables/
    ./uncalibrated/

The intent of the directories is as follows:

* `./data/` -- Directories containing data files corresponding to a single, unified observational data set.  Contents are calibrated images in sky brightness units (e.g., `K`, `MJY/SR`, `JY/PIX`, `W/M**2`)
* `./derived/` -- Derived science products from the data in `./data/`.  Examples would be star formation rates, gas surface density maps.
* `./docs/` -- Annoying documents like this one. 
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
4. Specified units using the `BUNIT` keyword.  Preference for MKS units as per FITS standard.
5. Specified resolution using `BMAJ`, `BMIN` and `BPA`.  Standard is to have `BMIN` and `BMAX` are in decimal degrees and `BPA` is in degrees east of north.
6. For spectral line data, the rest frequency in units specified by `RESTFRQ` in units of Hz.
7. Individual spectra should be grouped by galaxies and stored as FITS BINTABLES.  

These files represent sky quantities and have *not* been processed for specific scientific outcomes (e.g., diffuse 24 micron emission correction).

##### Naming Conventions 

Individual files should represent individual galaxies or continouous maps of groups (e.g., NGC 5194/5195).  Each file will be stored with their names given as the NED Preferred name for the object as the leading value.  Spaces are replaced with underscores: `_` and separated from the remainder of the file name by a `.`.  Consider the FITS data from the THINGS survey [`NGC_4826_NA_CUBE_THINGS.FITS`](http://www.mpia.de/THINGS/Data_files/NGC_4826_NA_CUBE_THINGS.FITS) should be stored as `MESSIER_064.NGC_4826_NA_CUBE_THINGS.FITS`.  Cleaving to this naming convention will help us traverse our data both by waveband and by galaxy.

The NED canonical name can be established directly by query NED by hand, using `astroquery` in python 

   from astroquery.ned import Ned
   (Ned.query_object('NGC 598'))['Object Name'].data.data[0]

Or the direct URL search, which returns parsable XML

    http://ned.ipac.caltech.edu/cgi-bin/nph-objsearch?%20extend=no&of=xml_main&objname=NGC+598
    
The preferred name is the first `TD` entry in the returned XML.  Multiple objects in the same file could be handled by soft links.

_Alternatively, we can just re-edit all the fits files to include the NED canonical name in the `OBJECT` keyword._

##### Uncertainty information

Since summary products of the data and understanding the quality of the data in each file requires understand the noise level, it would be ideal to add keyword information to the headers that captures the uncertainty in each file.  

##### Survey metadata information

Each directory in the `./data/` hierarchy will have a `SurveyName_README.txt` (e.g., `THINGS_README.txt` where `THINGS` is also the directory name.) file, which briefly explains where the data are from (URL!), what they represent, plus caveats and warnings if necessary.
