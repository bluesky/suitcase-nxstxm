=====
Usage
=====

Start by importing ``suitcase.nxstxm``.

.. code-block:: python

    import suitcase.nxstxm 

suitcase.nxstxm assumes basically the following sequence before it is called:

	1. The scan plan's metadata is constructed in the way described in :doc:`Metadata </metadata>`: **make_standard_metadata** 
	2. A **baseline** data stream exists in each scan that will end up as its own **entry** in the nxstxm file
	
	- for Sample Image Stacks there is one **baseline** data stream per: 

		- Spatial region
		
		- Polarization
	
	- for Tomography scans there is one **baseline** data stream per: 

		- Spatial region (there is only one spatial region supported currently)
		
		- Polarization
		
		- Goniometer Theta Angle
		
For intermediate scans within a plan that are NOT one of the above the only data steam is **primary**

	3. When the RunEngine has completed a scan plan, the uids that made up that plan are returned from the RunEngine.
	4. Those uids are then passed to a function that calls the nxstxm suitcase **export** function ex: of **do_data_export()** below, 
	it will call the export function in one of 3 ways based on the scan_type. In the example below the uids call **do_data_export** 
	and pass the uids and data directory where the export should be saved to.

.. code-block:: python
	
    def do_data_export(self, run_uids, datadir):
        '''
        executes inside a threadpool so it doesnt bog down the main event loop
        :return:
        '''
        print('do_data_export: waiting to execute export')
        #added a delay here so that in the case of an aborted/stopped scan the mongo db will have received all the
        #docs
        time.sleep(0.5)
        print('do_data_export: ok starting export')
        from cls.utils.file_system_tools import get_next_file_num_in_seq
        data_dir = self.active_user.get_data_dir()
        fprefix = 'C' + str(get_next_file_num_in_seq(data_dir, extension='hdf5'))

        scan_type = self.get_cur_scan_type()
        first_uid = run_uids[0]

        if(scan_type in [scan_types.SAMPLE_IMAGE_STACK, scan_types.TOMOGRAPHY_SCAN]):
            #could also just be multiple rois on a single energy
            self.do_multi_entry_export(run_uids, data_dir, fprefix)

        elif(scan_type in [scan_types.SAMPLE_POINT_SPECTRA, scan_type is scan_types.GENERIC_SCAN] ):
            self.do_point_spec_export(run_uids, data_dir, fprefix)

        else:
            for _uid in run_uids:
                print('starting basic export [%s]' % first_uid)
                header = MAIN_OBJ.engine_widget.db[first_uid]
                md = json.loads(header['start']['metadata'])
                docs = header.documents(fill=True)
                suit_nxstxm.export(docs, data_dir, file_prefix=fprefix, first_uid=first_uid)

        suit_nxstxm.finish_export(data_dir, fprefix, first_uid)



Basic Scan Export Types
=======================

For the following scan types **export** is just called directly, all the documents will be traversed and the nxstxm file will be saved as a
**.tmp** file while it is being constructed, this was done to prevent process' and widgets from a data collection application from trying to 
read the data as if it was completely written to the file from accidentily processing the file before it has completed contruction. To turn 
the **.tmp** file into the final **<filename>.hdf5** file is the job of **finish_export**, this function simply renames the .tmp file to the
final file name.



.. list-table::
   :widths: 15 10

   * - DETECTOR_IMAGE 
     - 0
   * - OSA_IMAGE
     - 1
   * - OSA_FOCUS
     - 2
   * - SAMPLE_FOCUS
     - 3
   * - SAMPLE_LINE_SPECTRA
     - 5
   * - SAMPLE_IMAGE
     - 6
   * - COARSE_IMAGE_SCAN
     - 9
   * - COARSE_GONI_SCAN
     - 10
     
Spectra Scans Export Types
==========================

When the RunEngine has completed it should return the uids that comprise the scan that just completed. For the following scan types

.. list-table::
   :widths: 15 10

   * - SAMPLE_POINT_SPECTRA  
     - 4
   * - GENERIC_SCAN
     - 8

.. code-block:: python
	
    def do_point_spec_export(self, run_uids, data_dir, fprefix):
        '''
        Point spec data is executed as a single run_uid, export is as an nxstxm entry
        :param run_uids:
        :param data_dir:
        :param fprefix:
        :return:
        '''
        #grab metadata from last run
        header = MAIN_OBJ.engine_widget.db[-1]
        first_uid = run_uids[0]
        print('starting point_spec export [%s]' % first_uid)
        header = MAIN_OBJ.engine_widget.db[first_uid]
        primary_docs = header.documents(fill=True)
        suit_nxstxm.export(primary_docs, data_dir, file_prefix=fprefix, first_uid=first_uid)
        


Sample Image Stack and Tomography Export Types
==============================================

When the RunEngine has completed it should return the uids that comprise the scan that just completed. For the following scan types

.. list-table::
   :widths: 15 10

   * - SAMPLE_IMAGE_STACK   
     - 7
   * - TOMOGRAPHY_SCAN 
     - 11
        
.. code-block:: python
	
    def do_multi_entry_export(self, run_uids, data_dir, fprefix):
        '''
        walk through a list of run_uids and export them as nxstxm entry's
        :param run_uids:
        :param data_dir:
        :param fprefix:
        :return:
        '''
        #grab metadata from last run
        header = MAIN_OBJ.engine_widget.db[-1]
        idx = 0
        for uid in run_uids:
            if(idx is 0):
                first_uid = uid
            idx += 1
        print('starting multi_entry export [%s]' % uid)
        for uid in run_uids:
            #export each uid as an nxstxm entry
            header = MAIN_OBJ.engine_widget.db[uid]
            primary_docs = header.documents(fill=True)
            suit_nxstxm.export(primary_docs, data_dir, file_prefix=fprefix, first_uid=first_uid)


Scan type lists for convienience
================================

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
            
	two_posner_scans = [scan_types.DETECTOR_IMAGE.value, scan_types.OSA_IMAGE.value, scan_types.COARSE_IMAGE_SCAN.value, \
	                    scan_types.COARSE_GONI_SCAN.value , scan_types.SAMPLE_IMAGE.value, scan_types.GENERIC_SCAN.value]
	single_entry_scans = [scan_types.DETECTOR_IMAGE.value, scan_types.OSA_IMAGE.value, scan_types.OSA_FOCUS.value , scan_types.SAMPLE_FOCUS.value , \
	                      scan_types.SAMPLE_LINE_SPECTRA.value, scan_types.SAMPLE_IMAGE.value , scan_types.COARSE_IMAGE_SCAN.value, \
						  scan_types.COARSE_GONI_SCAN.value, scan_types.GENERIC_SCAN.value]
	multi_entry_scans = [scan_types.SAMPLE_IMAGE_STACK.value , scan_types.SAMPLE_POINT_SPECTRA.value, scan_types.TOMOGRAPHY_SCAN.value]
	single_2d_scans = [scan_types.DETECTOR_IMAGE.value, scan_types.OSA_IMAGE.value, scan_types.COARSE_IMAGE_SCAN.value, \
	                           scan_types.COARSE_GONI_SCAN.value]
	focus_scans = [ scan_types.SAMPLE_FOCUS.value, scan_types.OSA_FOCUS.value]
	single_image_scans = [scan_types.SAMPLE_IMAGE.value]
	stack_type_scans = [scan_types.SAMPLE_IMAGE_STACK.value, scan_types.TOMOGRAPHY_SCAN.value]
	spectra_type_scans = [scan_types.SAMPLE_POINT_SPECTRA.value, scan_types.GENERIC_SCAN.value]
	line_spec_scans = [scan_types.SAMPLE_LINE_SPECTRA.value]