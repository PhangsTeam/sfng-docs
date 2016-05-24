### Data Store Specifications for the Star Formation in Nearby Galaxies Collaboration

This document outlines our data specification for the SFNG collaboration.  The data store fulfills several goals.

#### Objectives

1. Provide a single-point repository for science-ready data products to the collaboration.
2. Establish a stable data structure that can be algorthmically traversed to build up derived products.
3. Provide a method for release of data to the general community.
3. 

#### Structure

The data storage will be a single hierarchical file store.  Only certain file types should be stored in the directory tree.

    ./
    ./data_catalog.fits
    ./data/
    ./derived/
    ./docs/
    ./tables/
    ./uncalibrated/

The primary portion of the tree is the `./data/` directory structure, which will contain a series of subdirectories.  The major quantum of storage is a directory containing data files corresponding to a single, unified observational data set.

There is a single coarse organizational scheme to facilitate human navigation consisting of one layer of subdirectories that will enable traversal.  Some species are called out based on their utility in this research.  The 

    ./data/CO/
    ./data/HI/
    ./data/Ha/
    ./data/FIR/
    ./data/NIR/
    ./data/UV/
    ./data/Optical/
    ./data/Radio/
    
#### Naming Conventions 

Individual files will be stored with their names given as the NED Preferred name for the object as the leading value.  This can be established directly by query NED by hand, using `astroquery` in python 

   from astroquery.ned import Ned
   (Ned.query_object('NGC 598'))['Object Name'].data.data[0]

Or the direct URL search, which returns parsable XML

    http://ned.ipac.caltech.edu/cgi-bin/nph-objsearch?%20extend=no&of=xml_main&objname=NGC+598
    
The preferred name is the first `TD` entry in the returned XML.

#### Minimum file specifications for `./data/`

Data stored in the `./data/` tree are fully calibrated spectroscopy, images, and datacubes.  These should be science ready and carry with them a minimum of metadata to make them useful in the subsequent analysis.  Files will be included in the `./data/` tree if they fulfill all of the following specifications. 

1. FITS files strongly preferred.
2. Minimum valid WCS compliance that is readable in IDL via Astronomy library and in Python via astropy
3. Specified units using the `BUNIT` keyword
4. Specified resolution using `BMAJ`, `BMIN` and `BPA`
5. For spectral line data, the rest frequency in units specified by `RESTFRQ` 

These files represent sky quantities and have *not* been processed for specific scientific outcomes (e.g., diffuse 24 micron emission correction).
