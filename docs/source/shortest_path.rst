.. _shortest_path:

Shortest path
=============

We are building the shortest path for this sample of data::

        def shortest_path(stop_dict, trip_dict, t_c, srcs, the_moment, d=None,
                     patience=timedelta(hours=4)):
            M = datetime.max
            current = list()
            if d is None:
                d = dict()
                for k in stop_dict:
                    d[k] = (None, None, M)
                for src in srcs:
                    d[src] = (None, None, the_moment)
                    current.append(src)

            while len(current) > 0:
                next_level = list()
                for c_key in current:
                    cs = stop_dict[c_key]
                    limit_date = (d[c_key][2] + patience).date()
                    nt_d_t = cs.line_dijkstra(d[c_key][2], limit_date=limit_date)
                    for s_key in nt_d_t:
                        so = stop_dict[s_key]
                        (next_trip, d_t) = nt_d_t[s_key]
                        if d_t and d[s_key][2] > d_t:
                            d[s_key] = (cs, next_trip, d_t)
                            for ts in so.transfers:
                                t_key = (ts.name, ts.sn)
                                key = (t_key, s_key)
                                tt = timedelta(seconds=t_c[key])
                                d_t = d[s_key][2] + tt
                                if d[t_key][2] > d_t:
                                    d[t_key] = (so, None, d_t)
                                    if t_key not in next_level:
                                        next_level.append(t_key)
                                elif t_c[key] == 0 and t_key != c_key:
                                    if t_key not in next_level:
                                        next_level.append(t_key)
                current = sorted(next_level, key=lambda s_key: d[s_key][2])
            return d
