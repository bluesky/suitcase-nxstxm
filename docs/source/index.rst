.. Packaging Scientific Python documentation master file, created by
   sphinx-quickstart on Thu Jun 28 12:35:56 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive. 

suitcase.nxstxm Documentation
=============================

The nxstxm suitcase was developed at the Canadian Lightsource beamline 10ID1
and relies on metadata elements that may not exist at other synchrotron STXM's. 
This is a first pass at the documentation so it is quite likely that 
some key item will be missed. 


Metadata passed by all scan plans
=================================

The metadata dict that all scan plans pass is created by a single function so as to ensure
that all scans provide the same metadata. The link discusses what the metadata should look contain
as well in the **tests** directory of the main repo there is an **example_data** directory that contains 
a json file for every supported scan type so that if the documentation (that you are reading) is confusing
or inadequate you can refer to the json files to inspect what is actually passed.
		
.. toctree::
   :maxdepth: 2

   installation
   metadata
   usage
   
