========
Metadata
========

suitcase.nxstxm Metadata Dependancies
=====================================

There are a few dependancies in order for the the nxstxm export to function correctly, this
code was put together primarily to serve the pyStxm data acquisition project and then ported to
try and make it "universal" for other stxms to use, the truth though is that some of the metadata 
dependancies I have here may not make any sense to another facility, so there is likely going to
be some commenting out and additions added on your part to this code in order to adapt your stxm 
data collection application to use this suitcase.

The scans that have been tested with the nxstxm suitcase are enumerated below and used throughout 
the code to differentiate how the data should be handled.

   .. code-block:: python
   
		class scan_types(Enum):
				    DETECTOR_IMAGE = 0
				    OSA_IMAGE = 1
				    OSA_FOCUS = 2
				    SAMPLE_FOCUS = 3
				    SAMPLE_POINT_SPECTRA = 4
				    SAMPLE_LINE_SPECTRA = 5
				    SAMPLE_IMAGE = 6
				    SAMPLE_IMAGE_STACK = 7
				    GENERIC_SCAN = 8
				    COARSE_IMAGE_SCAN = 9
				    COARSE_GONI_SCAN = 10
				    TOMOGRAPHY_SCAN = 11
				    PATTERN_GEN_SCAN = 12


All stxm scans make a call to create a standard metadata dict as follows:

   .. code-block:: python
      
	    def make_standard_metadata(self, 
	    				entry_name, 
	    				scan_type, 
	    				primary_det=DNM_DEFAULT_COUNTER, 
	    				override_xy_posner_nms=False):
	        '''
	        return a dict that is standard for all scans and that gives the 
	        data suitcase all it needs to be able to save the nxstxm datafile
	
	        :param entry_name: The nexus entry name to use for this scan ex: entry0
	        :param scan_type:	An enumeration value of type defined above
	        :param primary_det: The name of the primary detector used to record the main stxm 
	        		data, ex: counter0
	        :param override_xy_posner_nms: I cant remember what this is, will fill it in in 
	        		the future
	        :return:

        '''
	
	        dct = {}
	        dct['entry_name'] = entry_name
	        dct['scan_type'] = scan_type
	        dct['sp_id_lst'] = self._master_sp_id_list
	        dct['rois'] = self.get_rois_dict(override_xy_posner_nms)
	
	        #dct['device_reverse_lu_dct'] = self.main_obj.get_device_reverse_lu_dct()
	        dct['dwell'] = self.dwell
	        dct['primary_det'] = primary_det
	        dct['zp_def'] = self.get_zoneplate_info_dct()
	        dct['wdg_com'] = dict_to_json(self.wdg_com)
	        dct['img_idx_map'] = dict_to_json(self.img_idx_map['%d' % self._current_img_idx])
	        dct['rev_lu_dct'] = self.main_obj.get_device_reverse_lu_dct()
	        return(dct)


**metadata dict**

.. list-table::
   :widths: 15 85

   * - dct['entry_name'] 
     - **str** - entry_name like **entry0**
   * - dct['scan_type']
     - **int** - scan_type of the enumerated types listed above
   * - dct['sp_id_lst']
     - **list** - is a list of the spatial id's that make up this scan ex: [0,1,2]
   * - dct['rois']
     - **dict of dicts** - a dictionary thats key is the spatial id's for this scan, each spatial dct['roi'][<spatial_id>] contains roi dicts for ['X', 'Y', 'Z', 'GONI', 'EV_ROIS', 'ZP', 'OSA']
   * - dct['dwell']
     - **float** - the dwell time for the scan
   * - dct['primary_det']
     - **str** - name of the primary plotted detector ex: **counter0**
   * - dct['zp_def']
     - **dict** - a **zoneplate definition** dict that contains the currently installed zoneplate parameters
   * - dct['wdg_com']
     - **dict** - a **widget communications** dict
   * - dct['img_idx_map']
     - **dict** - a **img_idx_map** dict that's key is the iteration number of the scan and contains the followng fields
   * - dct['rev_lu_dct']       
     - **dict** - a **reverse lookup** dictionary that contains descriptive device names and the control system source names

**zoneplate definition dict**	

	.. code-block:: python
	
		{'fl': -3955.3002,		# focal length
		 'zpD': 250.0,		# diameter
		 'zpCStop': 100.0,		# central stop
		 'zpOZone': 25.0,		# outer zone diameter
		 'zpA1': -5.067,		# A1 slope
		 'zp_idx': 7}		# enumeration of the zoneplate


**img_idx_map dict**	

This is a dictionary that maps each iteration of the scan so that once the scan is completed the data for each iteration can correctly be placed in the 
right entry, polarization (each polaraization is treated as its own entry like a spatial ID), and energy index into the data array (ev, y, x) 

	.. code-block:: python	
	
		{"0": {
			"e_idx": 0, 
			"entry": "entry0", 
			"sp_id": 0, # the 2D data for iteration 0 of scan is placed in entry0->counter0->data[0]
			"pol_idx": 0, 
			"sp_idx": 0},
		{"1": {
			"e_idx": 1, 
			"entry": "entry0", 
			"sp_id": 0, # the 2D data for iteration 1 of scan is placed in entry0->counter0->data[1]
			"pol_idx": 0, 
			"sp_idx": 0}
		...
		{"7": {
			"e_idx": 3, 
			"entry": "entry1", 
			"sp_id": 1, # the 2D data for iteration 7 of scan is placed in entry1->counter0->data[3]
			"pol_idx": 0, 
			"sp_idx": 1}
		...
		'131': {
			'e_idx': 32,
			'entry': 'entry3'
			'sp_id': 1, # the 2D data for iteration 131 of scan is placed in entry3->counter0->data[32]
		  'pol_idx': 1,
		  'sp_idx': 1}
	  ...
	  	you get the idea

**widget communications dict**	

	.. code-block:: python
	
		{'SINGLE_LST': **single_list_dict**
		}		# enumeration of the zoneplate

**single_list_dict**	

	.. code-block:: python
	
		{'SP_ROIS': list of **spatial region of interest** dicts,
		'POL_ROIS', list of polarization dicts 
		'DWELL', list of the dwell times
		'EV_ROIS'] list of energy values for the scan
		}

**reverse lookup dict**	

A reverse lookup dictionary used to get the devices source name from the descriptive name

	.. code-block:: python
	
		{'SampleFineX': 'IOC:m100',
		 'SampleFineY': 'IOC:m101',
		 'Coarse.X': 'IOC:m112',
		 'Coarse.Y': 'IOC:m113',
		 'OSA.X': 'IOC:m104',
		 'OSA.Y': 'IOC:m105',
		 'OSA.Z': 'IOC:m106C',
		 ...


**spatial region of interest dict**	

There is a spatial roi dict for the following axis' 'X', 'Y', 'Z'. The following axis 'GONI', 'OSA', 'ZP' are all dicts that contain each also contain spatial roi dicts for X,Y and Z
on the following way:

 Goniometer spatial roi dicts: ['GONI']['X'], ['GONI']['Y'], ['GONI']['Z'], ['GONI']['THETA'] 
  
 Order Sorting Aperature spatial roi dicts: ['OSA']['X'], ['OSA']['Y'], ['OSA']['Z']
 
 Zoneplate spatial roi dicts: ['ZP']['X'], ['ZP']['Y'], ['ZP']['Z']


The spatial region of interest dictionarys look like the following example for ['X']
	.. code-block:: python
	
		{'NAME': 'GONI.X',
		 'ID': 'BASE_ROI',
		 'ID_VAL': -1,
		 'CENTER': 588.4146077203648,
		 'RANGE': 10.747720364741667,
		 'NPOINTS': 50,
		 'ENABLED': True,
		 'IS_POINT': False,
		 'OFFSET': 0.0,
		 'STEP': 0.21934123193350574,
		 'START': 583.0407475379939,
		 'STOP': 593.7884679027356,
		 'SETPOINTS': array([583.04074754, 583.26008877, 583.47943   , 583.69877123,
		        583.91811247, 584.1374537 , 584.35679493, 584.57613616,
		        584.79547739, 585.01481863, 585.23415986, 585.45350109,
		        585.67284232, 585.89218355, 586.11152479, 586.33086602,
		        586.55020725, 586.76954848, 586.98888971, 587.20823094,
		        587.42757218, 587.64691341, 587.86625464, 588.08559587,
		        588.3049371 , 588.52427834, 588.74361957, 588.9629608 ,
		        589.18230203, 589.40164326, 589.6209845 , 589.84032573,
		        590.05966696, 590.27900819, 590.49834942, 590.71769066,
		        590.93703189, 591.15637312, 591.37571435, 591.59505558,
		        591.81439682, 592.03373805, 592.25307928, 592.47242051,
		        592.69176174, 592.91110298, 593.13044421, 593.34978544,
		        593.56912667, 593.7884679 ]),
		 'POSITIONER': 'GoniX',
		 'SRC': None,
		 'TOP_LEVEL': False, 
		 'DATA_LEVEL': False,
		 'SCAN_RES': 'COARSE'}


		
.. toctree::
   :maxdepth: 2

   
