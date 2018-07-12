---
layout: post
title: "expdp hung in oracle12c"
date: 2018-07-09
---
Q - expdp hung on “wait for unread message on broadcast channel”  
  
investigation -   
expdp $WSP_SCHEMA_USR/$WSP_SCHEMA_PWD directory=streams_pump_dir parfile=tables.par parallel=10 logfile=expdp_strminst.log exclude=CONSTRAINT,INDEX,TRIGGER dumpfile=streams_export.dmp FLASHBACK_TIME=$flashback_time job_name=STRMINST reuse_dumpfiles=Y content=ALL network_link=$DBNAME 2
  
add TRACE=480300 in command line,  
trace was generated on dm process, it stucks at a system call:   
select sql_text from v$sqlarea where sql_id  = '65sh6sucvcnc8';  
SQL_TEXT  
----------------------------------------------------------------  
BEGIN    SYS.KUPM$MCP.MAIN('STRMINST', 'R6', 0, 4719360);  END;  

Note that parallel, FLASHBACK_TIME are the 2 features involved in some bugs,   
tried to remove parallel, same error   
remove flashback_time, expdp works fine   
  
Here expdp is used to copy data from a master site, setup a base for streams replication.   
flashback_time is to perform a time consistent export, without it, what will be the side affect?   
-- no side effect, provided we setting streams instantiation SCN earlier than the starting time of export.   

 
