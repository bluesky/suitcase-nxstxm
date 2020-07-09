
# suitcase.nxstxm

This is a suitcase subpackage for writing a particular file format.

## Installation

In order for the nxstxm package to be correctly built and installed into the directory

    lib/site-packages/suitcase/nxstxm

the build must be performed by typing the following on the command line from inside the repo directory:
```
pip install .
```

# suitcase.nxstxm Documentation

The nxstxm suitcase was developed at the Canadian Lightsource beamline 10ID1 and relies on metadata 
elements that may not exist at other synchrotron STXM’s. This is a first pass at the documentation 
so it is quite likely that some key item will be missed.

# Metadata passed by all scan plans

The metadata dict that all scan plans pass is created by a single function so as to ensure that all 
scans provide the same metadata. The link discusses what the metadata should look contain as well in 
the tests directory of the main repo there is an example_data directory that contains a json file for
 every supported scan type so that if the documentation (that you are reading) is confusing or 
 inadequate you can refer to the json files to inspect what is actually passed.
 
 ```
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
```
 
 All stxm scans make a call to create a standard metadata dict as follows:
 ```
    def make_standard_metadata(self,
                                entry_name,
                                scan_type,
                                primary_det=DNM_DEFAULT_COUNTER,
                                override_xy_posner_nms=False):
        """
        return a dict that is standard for all scans and that gives the
        data suitcase all it needs to be able to save the nxstxm datafile

        :param entry_name: The nexus entry name to use for this scan ex: entry0
        :param scan_type:       An enumeration value of type defined above
        :param primary_det: The name of the primary detector used to record the main stxm
                        data, ex: counter0
        :param override_xy_posner_nms: I cant remember what this is, will fill it in in
                        the future
        :return:

        """

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
```

## metadata dict
```
    dct[‘entry_name’] 	str - entry_name like entry0
    dct[‘scan_type’] 	int - scan_type of the enumerated types listed above
    dct[‘sp_id_lst’] 	list - is a list of the spatial id’s that make up this scan ex: [0,1,2]
    dct[‘rois’] 	dict of dicts - a dictionary thats key is the spatial id’s for this scan, each spatial dct[‘roi’][<spatial_id>] 
                                    contains roi dicts for [‘X’, ‘Y’, ‘Z’, ‘GONI’, ‘EV_ROIS’, ‘ZP’, ‘OSA’]
    dct[‘dwell’] 	float - the dwell time for the scan
    dct[‘primary_det’] 	str - name of the primary plotted detector ex: counter0
    dct[‘zp_def’] 	dict - a zoneplate definition dict that contains the currently installed zoneplate parameters
    dct[‘wdg_com’] 	dict - a widget communications dict
    dct[‘img_idx_map’] 	dict - a img_idx_map dict that’s key is the iteration number of the scan and contains the followng fields
    dct[‘rev_lu_dct’] 	dict - a reverse lookup dictionary that contains descriptive device names and the control system source names
 ```

## zoneplate definition dict
```
    {'fl': -3955.3002,      # focal length
     'zpD': 250.0,          # diameter
     'zpCStop': 100.0,      # central stop
     'zpOZone': 25.0,       # outer zone diameter
     'zpA1': -5.067,        # A1 slope
     'zp_idx': 7}           # enumeration of the zoneplate
```

## img_idx_map dict
This is a dictionary that maps each iteration of the scan so that once the scan is completed the 
data for each iteration can correctly be placed in the right entry, polarization (each 
polaraization is treated as its own entry like a spatial ID), and energy index into the data array 
(ev, y, x)

```
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
```

## widget communications dict

```
    {'SINGLE_LST': **single_list_dict**}               
```

## single_list_dict
```
    {
        'SP_ROIS': list of **spatial region of interest** dicts,
        'POL_ROIS', list of polarization dicts
        'DWELL', list of the dwell times
        'EV_ROIS'] list of energy values for the scan
    }
```

## reverse lookup dict
A reverse lookup dictionary used to get the devices source name from the descriptive name

```
    {'SampleFineX': 'IOC:m100',
     'SampleFineY': 'IOC:m101',
     'Coarse.X': 'IOC:m112',
     'Coarse.Y': 'IOC:m113',
     'OSA.X': 'IOC:m104',
     'OSA.Y': 'IOC:m105',
     'OSA.Z': 'IOC:m106C',
     ...}
```

## spatial region of interest dict

There is a spatial roi dict for the following axis’ ‘X’, ‘Y’, ‘Z’. The following axis ‘GONI’, 
‘OSA’, ‘ZP’ are all dicts that contain each also contain spatial roi dicts for X,Y and Z on the 
following way:

    Goniometer spatial roi dicts: [‘GONI’][‘X’], [‘GONI’][‘Y’], [‘GONI’][‘Z’], [‘GONI’][‘THETA’]

    Order Sorting Aperature spatial roi dicts: [‘OSA’][‘X’], [‘OSA’][‘Y’], [‘OSA’][‘Z’]

    Zoneplate spatial roi dicts: [‘ZP’][‘X’], [‘ZP’][‘Y’], [‘ZP’][‘Z’]

The spatial region of interest dictionarys look like the following example for [‘X’]

```
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

```

# Usage

Start by importing suitcase.nxstxm.
```
import suitcase.nxstxm
```

suitcase.nxstxm assumes basically the following sequence before it is called:

        1. The scan plan’s metadata is constructed in the way described in Metadata: make_standard_metadata
        2. A baseline data stream exists in each scan that will end up as its own entry in the nxstxm file

            for Sample Image Stacks there is one baseline data stream per:
    
                    Spatial region
                    Polarization

            for Tomography scans there is one baseline data stream per:
    
                    Spatial region (there is only one spatial region supported currently)
                    Polarization
                    Goniometer Theta Angle

            For intermediate scans within a plan that are NOT one of the above the only data steam is primary

        3. When the RunEngine has completed a scan plan, the uids that made up that plan are returned from the RunEngine.

        4. Those uids are then passed to a function that calls the nxstxm suitcase export function ex: of do_data_export() 
            below, it will call the export function in one of 3 ways based on the scan_type. In the example below the uids 
            call do_data_export and pass the uids and data directory where the export should be saved to.
```
    def do_data_export(self, run_uids, datadir):
        """
        executes inside a threadpool so it doesnt bog down the main event loop
        :return:
        """
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
```


### Basic Scan Export Types

For the following scan types export is just called directly, all the documents will be traversed and the nxstxm file 
will be saved as a .tmp file while it is being constructed, this was done to prevent process’ and widgets from a data 
collection application from trying to read the data as if it was completely written to the file from accidentily 
processing the file before it has completed contruction. To turn the .tmp file into the final <filename>.hdf5 file is 
the job of finish_export, this function simply renames the .tmp file to the final file name.

    DETECTOR_IMAGE 	0
    OSA_IMAGE 	1
    OSA_FOCUS 	2
    SAMPLE_FOCUS 	3
    SAMPLE_LINE_SPECTRA 	5
    SAMPLE_IMAGE 	6
    COARSE_IMAGE_SCAN 	9
    COARSE_GONI_SCAN 	10
    PATTERN_GEN_SCAN = 12


### Spectra Scans Export Types

When the RunEngine has completed it should return the uids that comprise the scan that just completed. For the following scan types

    SAMPLE_POINT_SPECTRA 	4
    GENERIC_SCAN 	8

```
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
```
### Sample Image Stack and Tomography Export Types

When the RunEngine has completed it should return the uids that comprise the scan that just completed. For the following 
scan types

    SAMPLE_IMAGE_STACK 	7
    TOMOGRAPHY_SCAN 	11
    PTYCHOGRAPHY_SCAN = 13
```
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
```

### Scan type lists for convienience

```
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
        PTYCHOGRAPHY_SCAN = 13

two_posner_scans = [scan_types.DETECTOR_IMAGE.value, scan_types.OSA_IMAGE.value, scan_types.COARSE_IMAGE_SCAN.value, \
                    scan_types.COARSE_GONI_SCAN.value , scan_types.SAMPLE_IMAGE.value, scan_types.GENERIC_SCAN.value]
single_entry_scans = [scan_types.DETECTOR_IMAGE, scan_types.OSA_IMAGE, scan_types.OSA_FOCUS , scan_types.SAMPLE_FOCUS , \
                      scan_types.SAMPLE_LINE_SPECTRA , scan_types.SAMPLE_IMAGE , scan_types.COARSE_IMAGE_SCAN, \
					  scan_types.COARSE_GONI_SCAN, scan_types.GENERIC_SCAN, scan_types.PATTERN_GEN_SCAN]
multi_entry_scans = [scan_types.SAMPLE_IMAGE_STACK , scan_types.SAMPLE_POINT_SPECTRA, scan_types.TOMOGRAPHY_SCAN]
single_2d_scans = [scan_types.DETECTOR_IMAGE, scan_types.OSA_IMAGE, scan_types.COARSE_IMAGE_SCAN, \
                           scan_types.COARSE_GONI_SCAN]
focus_scans = [ scan_types.SAMPLE_FOCUS, scan_types.OSA_FOCUS]
single_image_scans = [scan_types.SAMPLE_IMAGE, scan_types.PATTERN_GEN_SCAN]
stack_type_scans = [scan_types.SAMPLE_IMAGE_STACK, scan_types.TOMOGRAPHY_SCAN, scan_types.PTYCHOGRAPHY_SCAN]
spectra_type_scans = [scan_types.SAMPLE_POINT_SPECTRA, scan_types.GENERIC_SCAN]
line_spec_scans = [scan_types.SAMPLE_LINE_SPECTRA]
```


## Documentation

See the [suitcase documentation](https://blueskyproject.io/suitcase).