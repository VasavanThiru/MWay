.. _project_classes:

===============
Project Classes
===============


-------
Station
-------

Class Station contains id, name, latitude, longitude, short name of stations::

        class Station :
        def __init__(self, ids, name=None, lat=None, lon=None,sn=None,
                 nexts=None, transfers=None, date_trips=dict())


------
Voyage
------

Class Voyage joins stations of each trips.::

        class Voyage():
            def __init__(self, trip_id, dates=list(), stop_times=None)
