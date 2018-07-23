# Tutorial


## Initialize


The first step is to import the ONE class. In the IBL case, the class has to be instantiated: behind the scenes, the constructor connects to the Alyx database and gets credentials. The connections settings are defined in the *params.py* and the *params_secret.py* files.

```python
from one_ibl.one import ONE
myone = ONE() # need to instantiate the class to have the API.
```

## Load method
### General Use
The IBL uses sessions UUID as per the Alyx database as experiments ID.
If the EEID is known, one can access directly the numpy arrays this way:
```python
dataset_types = ['cwStimOn.times', 'cwStimOn.contrastRight', 'cwStimOn.contrastLeft']
eid = 'http://localhost:8000/sessions/698361f6-b7d0-447d-a25d-42afdef7a0da'
t, cr, cl = myone.load(eid, dataset_types=dataset_types)
```

Depending on the use case, it may be handier to wrap the arrays in a dictionary
(a structure for Matlab users) so that a bit of context is included with the array. This could be useful for custom format, or if the user wants to re-access the files locally:
```python
my_dict = myone.load(eid, dataset_types=dataset_types,  dict_output=True)
list(my_dict.keys())
```
```python
['data', 'id', 'local_path', 'dataset_type', 'url', 'eid']
```
The dictionary contains the following keys, each of which contains 3 lists corresponding the the 3 queried datasets

-   data (*numpy.array*): the numpy array
-   id (*str*): the UUID of the dataset in Alyx
-   local_path (*str*): the local full path of the file
-   dataset_type (*str*): as per Alyx table
-   url (*str*): the link on the FlatIron server
-   eid (*str*): the session UUID in Alyx

It is also possible to query all datasets attached to a given session, in which case
the output has to be a dictionary:
```python
my_dict = myone.load(eid)
```

### Specific cases
If a dataset type queried doesn't exist or is not on the FlatIron server, an empty list
is returned. This allows to keep the proper order of output arguments
```python
dataset_types = ['cwStimOn.times', 'thisDataset.IveJustMadeUp', 'cwStimOn.contrastLeft']
t, cr, cl = myone.load(eid, dataset_types=dataset_types)
```
Returns an empty list for *cr* so that *t* and *cl* still get assigned the proper values.

## List method
The methods allow to access 3 tables of the current database:
-   dataset-type
-   users
-   subjects

For example to print a list of the dataset-types in the command window:
```python
myone.list(table='dataset-types', verbose=True)
```

One can also select several fields

```python
from one_ibl.misc import pprint
list_types , dtypes = myone.list(table=['dataset-types','users'])
pprint(list_types)
pprint(dtypes)
```
This will give the following output:
```
[
    "Block",
    "cwFeedback.rewardVolume",
    "cwFeedback.times",
    "cwFeedback.type",
    "cwGoCue.times",
    "cwResponse.choice",
    "cwResponse.times",
    "cwStimOn.contrastLeft",
    "cwStimOn.contrastRight",
    "cwStimOn.times",
    "cwTrials.inclTrials",
    "cwTrials.intervals",
    "cwTrials.repNum",
    "expDefinition",
    "galvoLog",
    "Hardware Info",
    "lfp.raw",
    "Parameters",
    "photometry.calciumLeft_normalized",
    "photometry.calciumRight_normalized",
    "photometry.timestamps",
    "unknown",
    "wheel.position",
    "wheel.timestamps",
    "wheel.velocity"
    ],
    [
        "miles",
        "i-chun",
        "lauren",
        "cyrille",
        "marius",
        "julien",
        "matteo",
        "julie",
        "mush",
        "daisuke",
        "Philip",
        "claire",
        "carsen",
        "peter",
        "Stephane",
        "Anna",
        "max",

[
    {
        "name": "Block",
        "created_by": "miles",
        "description": "",
        "filename_pattern": "*_Block.*"
    },
    {
        "name": "cwFeedback.rewardVolume",
        "created_by": "miles",
        "description": "Size of the water reward given in microlitres",
        "filename_pattern": "cwFeedback.rewardVolume.*"
    },
    # and so on...
```


## Search method
The search methods allows to query the database to filter the list of UUIDs according to
the following fields:
-   dataset_types
-   users
-   subject
-   date_range

Here is the simple implementation of the filter, where we query for the EEIDs (sessions) co-owned by
all of the following users: Morgane, miles and armin (case-sensitive).
```python
sl , sd =  myone.search(users=['Morgane','miles','armin'])
pprint(sl)
```

The following would get all of the dataset of which Morgane is the owner or co-owner:
```python
sl , sd =  myone.search(users=['Morgane'])
pprint(sl)
```


