---
layout: post
title: "Oracle Streams Checkpoint retention"
date: 2018-07-13
---

dba_capture.REQUIRED_CHECKPOINT_SCN was used to tell if an archived redo log still need for Streams replication.   
After a network issue, We need RMAN backup job to remove stale redo log immediately due to disk pressure.   
By default, Streams checkpoints every 6 hours, to manually decrease this parameter temporarily:   

exec dbms_propagation_adm.stop_propagation('DB1_PROP');  
execute dbms_capture_adm.stop_capture('DB1_CAP');   

-- save original value for backup, 21600 / 60 = 360 min = 6 hours , the checkpoint SCN will be updaed every 6 hours   
--- execute DBMS_CAPTURE_ADM.SET_PARAMETER('DB1_CAP', '_CKPT_RETENTION_CHECK_FREQ', '21600');     
-- New set to 10s to force the update on the view then change it back to 21600 to leave as it was before *default config  
execute DBMS_CAPTURE_ADM.SET_PARAMETER('DB1_CAP', '_CKPT_RETENTION_CHECK_FREQ', '10');    
  
exec dbms_propagation_adm.start_propagation('DB1_PROP');    
execute dbms_capture_adm.start_capture('DB1_CAP');    



