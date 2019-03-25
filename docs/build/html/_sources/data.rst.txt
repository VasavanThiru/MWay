.. _data:

====
Data
====

The data is given by the RATP in `GTFS <https://developers.google.com/transit/gtfs/reference/?hl=fr/>`_  format.

All these files are organized like that for each line of subway :

* agency.txt
* calendar.txt
* calendar_dates.txt
* routes.txt
* stops.txt
* stop_times.txt
* transfers.txt
* trips.txt



Use of Data
===========

We decided to merge all the file in a great file named METRO, it contains line **1 to 14** of subway::



        import pandas as pd
        import os

        os.getcwd()
        list_metro=os.listdir("METRO")
        print(list_metro)
        stops={}
        routes={}
        transfers={}
        calendar={}
        calendar_dates={}
        stop_times={}
        trips={}
        for i in list_metro:
            stops[i]=pd.read_csv(dossier+i+'/stops.txt', header = 0,usecols=['stop_id','stop_name','stop_desc','stop_lat','stop_lon'])
            routes[i] = pd.read_csv(dossier+i+'/routes.txt', header = 0,usecols=['route_id','route_short_name','route_type'])
            transfers[i] = pd.read_csv(dossier+i+'/transfers.txt', header = 0,usecols=['from_stop_id','to_stop_id','min_transfer_time'])
            calendar_dates[i] = pd.read_csv(dossier+i+'/calendar_dates.txt', header = 0)
            calendar[i] = pd.read_csv(dossier+i+'/calendar.txt', header = 0,usecols=['service_id'])
            stop_times[i] = pd.read_csv(dossier+i+'/stop_times.txt', header = 0, usecols=['trip_id','departure_time','stop_id','stop_sequence'])
            trips[i] = pd.read_csv(dossier+i+'/trips.txt', header = 0,usecols=['route_id','service_id','trip_id','direction_id'])
