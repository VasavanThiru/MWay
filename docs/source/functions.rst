.. _functions

Functions
=========


In this part we initially build all the functions in order to organize the previous file.

------------------
Structuring Data
------------------

We are building all dictionary and stuctures in ``Structuring_Data``::

        def Structuring_Data(calendar,calendar_dates,stops,stop_times,trips,transfers,list_metro):


            dict_serv_id={}
            for l in list_metro:
                liste_s_id=list()
                for i in calendar[l]['service_id']:
                    liste_s_id.append(i)
                    dict_serv_id[i]=[]
                j=0
                for i in calendar_dates[l]['date']:
                    i=str(i)
                    d=date(int(i[0:4]),int(i[4:6]),int(i[6:8]))
                    dict_serv_id[calendar_dates[l]['service_id'][j]].append(d)
                    j=j+1

            dict_route = {}
            for l in list_metro:
                j=0
                for i in routes[l]['route_id']:
                    dict_route[i]=(int(routes[l]['route_type'][j]),str(routes[l]['route_short_name'][j]))
                    j=j+1
            del l,j,i


            dict_trips = {}
            for l in list_metro:
                j=0
                for i in trips[l]['trip_id']:
                    dict_trips[i]=(trips[l]['route_id'][j],trips[l]['service_id'][j])
                    j=j+1


            dict_stp_t={}
            for l in list_metro:
                for j in range(1,len(stop_times[l]['trip_id'])):
                    stop_id=stop_times[l]['stop_id'][j]
                    if dict_stp_t.get(stop_id) is None :
                        trip_id = stop_times[l]['trip_id'][j]
                        if trip_id in dict_trips :
                            dict_stp_t[stop_id]=trip_id
            del l , j


            dict_stations={}
            start_time_6 = time.time()
            for l in list_metro:
                j=0
                for i in stops[l]['stop_id']:
                    dict_stations[i]=(stops[l]['stop_name'][j],str(routes[l]['route_short_name'][0]))
                    j=j+1
            del i,j



            transfer_cost = dict()
            for l in list_metro:
                for j in range(len(transfers[l]['from_stop_id'])):
                    dep_trsf = transfers[l]['from_stop_id'][j]
                    arr_trsf = transfers[l]['to_stop_id'][j]
                    from_stop_id = dict_stations.get(dep_trsf)
                    to_stop_id = dict_stations.get(arr_trsf)
                    if from_stop_id and to_stop_id and\
                            transfer_cost.get((from_stop_id, to_stop_id)) is None:
                        cost = transfers[l]['min_transfer_time'][j]
                        transfer_cost[(from_stop_id, to_stop_id)] = cost
                        transfer_cost[(to_stop_id, from_stop_id)] = cost
            del j


            return dict_serv_id,dict_route,dict_trips,dict_stp_t,dict_stations,transfer_cost

-----------------------
Build Object stations
-----------------------


``Build_Object_stations`` make the Stations objects from Stations class ::

        def Build_Object_stations(stops,stop_times,routes,list_metro,dict_stations):

            dict_obj_stations={}
            sn_date_trips = dict()
            for l in list_metro:
                for j in range(len(stops[l]['stop_id'])):
                            stop_id = int(stops[l]['stop_id'][j])
                            trip_id = dict_stp_t[stop_id]
                            route_id = dict_trips[trip_id][0]
                            sn = dic_route[route_id][1]
                            stop_key = (stops[l]['stop_name'][j], sn)
                            so = dict_obj_stations.get(stop_key)
                            if so is None:
                                date_trips = sn_date_trips.get(sn)
                                if date_trips is None:
                                    date_trips = dict()
                                    sn_date_trips[sn] = date_trips
                                dict_obj_stations[stop_key] = classes.Station([stop_id],stops[l]['stop_name'][j],stops[l]['stop_lat'][j],
                                                         stops[l]['stop_lon'][j],sn,
                                                         date_trips=date_trips)
                            else:
                                so.ids.append(stop_id)
            return dict_obj_stations


-------------------
Build Object trip 
-------------------

``Build Object trip`` is function that builds Trips objects ::

        def Build_Object_trips(stop_times,dict_obj_stations,dict_stations,dict_trips,dict_serv_id):
            """Load the file stop_times.txt and return a dict."""

            ld_trip = dict()

            for l in list_metro:
                n=(len(stop_times[l]['trip_id']))
                r_i = stop_times[l]['stop_id'][0]
                r_key = dict_stations.get(r_i)
                trip_id = stop_times[l]['trip_id'][0]
                service_id = dict_trips[trip_id][1]
                dates = dict_serv_id [service_id]
                ld_trip[trip_id] = classes.Voyage(trip_id, dates)
                sorted(ld_trip[trip_id].dates)
                prev_trip_id = trip_id
                p_key = r_key
                for j in range(1,n):
                    r_i = stop_times[l]['stop_id'][j]
                    r_key = dict_stations.get(r_i)
                    dict_obj_stations[r_key]
                    trip_id = stop_times[l]['trip_id'][j]
                    if prev_trip_id != trip_id:
                        sorted(ld_trip[prev_trip_id].dates)
                        service_id = dict_trips[trip_id][1]
                        dates = dict_serv_id [service_id]
                        ld_trip[trip_id] = classes.Voyage(trip_id, dates)
                    else:
                        if dict_obj_stations[r_key] not in dict_obj_stations[p_key].nexts:
                            dict_obj_stations[p_key].nexts.append(dict_obj_stations[r_key])

                    horaire = stop_times[l]['departure_time'][j].split(':')
                    ld_trip[trip_id].stations_trip.append((dict_obj_stations[r_key],
                                                         (int(horaire[0]), int(horaire[1]),
                                                          int(horaire[2]))))
                    p_key = r_key
                    prev_trip_id = trip_id

            return ld_trip


--------------------------
Add trip to Object Station
--------------------------

``add_trip``::

        def add_trips(ld_trip):
            """Add trips to each stop."""
            for Object_Voyage in ld_trip.values():
                Object_Station = Object_Voyage.stations_trip[0][0]

                for date in Object_Voyage.dates:
                    trips = Object_Station.date_trips.get(date)
                    if trips is None:
                        trips = list()
                        Object_Station.date_trips[date] = trips
                    if Object_Voyage not in trips:
                        trips.append(Object_Voyage)

----------------
Adding transfers
----------------

``nexts_transfers``::

        def nexts_transfers(stop_dict, t_c):
            """Add connections and some zero-transfers."""

            for Object_Station in stop_dict.values():
                for ns in Object_Station.nexts:
                        # TODO: use len for precision
                    if Object_Station not in ns.nexts:
                        ns.nexts.append(Object_Station)
                            #print(ns.nexts)
                    if len(Object_Station.ids) > 1:
                        s_key = (Object_Station.name, Object_Station.sn)
                        key = (s_key, s_key)
                        if t_c.get(key) is None:
                            t_c[key] = 0

            for s_key in stop_dict:
                Object_Station = stop_dict[s_key]
                if len(Object_Station.nexts) > 2:
                    if Object_Station not in Object_Station.transfers:
                        Object_Station.transfers.append(Object_Station)
                    key = (s_key, s_key)
                    if t_c.get(key) is None:
                        t_c[key] = 0
