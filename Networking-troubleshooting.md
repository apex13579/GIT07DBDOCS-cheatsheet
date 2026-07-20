# ospf troubleshooting
- it all starts with the neighbor process
  1. ospf will start out with a hello message and that is to see if they are compatible (same subnet mask), in the same area, have the same hello and dead interval
  2. then check dbd (database description)
  3. then check lsa (link state advertisement) has the information on one network 
  4. then check lsu (link state update) has the updates for the networks 
  5. then check lsack (link state acknoledgement) confirms receiving
 
- top show commands in order
  1. show ip ospf neighbor
  2. show ip ospf database
  3. show ip ospf process
  4. router-id
  5. show ospf int brief
  6. show ip protocols
 
- states
  1. down = nothing happening
  2. init = (initiation) where the hello messages are sent out
  3. 2-way = non managerial neighbor relationship
  4. exstart = looks at database and exchange summeries with the dr or bdr
  5. exchange = looks at lsa and lsu to ensure info is correct
  6. loading = 
  7. full =
 
- common troubleshooting delemas
|issue|check|
|---|---|
|flapping network from down to init|check the data because something in the compatibility lisy is wrong|
