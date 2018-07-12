---
layout: post
title: "create instance in OpenStack Queens got - 500 Internal Server Error"
date: 2018-07-06
---

Q -- create instance on openstack queens got error "500 Internal Server Error"  
openstack server create --image Ubuntu_16.04_LTS --flavor ... --key-name ... --nic net-id=.... ttt  
openstack server list  
  
investigation -- due to syntax error in nova.conf  
/var/log/nova/nova-scheduler.log shows 500 internel server error :   
  *The server encountered an internal error or misconfiguration and was unable to complete your request. 
  Please contact the server administrator at [no address given] to inform them of the time this error occurred,  and the actions you performed just before this error.    
   More information about this error may be available in the server error log.*      

go to server error log, /var/log/httpd/placement_wsgi_error.log, First error shows at Jul 08:   
 *[Sun Jul 08 03:44:39[:error]  mod_wsgi (pid=14961): Target WSGI script '/var/www/cgi-bin/nova/nova-placement-api' cannot be loaded as Python module.  
  [Sun Jul 08 03:44:39[:error]  mod_wsgi (pid=14961): Exception occurred processing WSGI script '/var/www/cgi-bin/nova/nova-placement-api'.  
......
  [Sun Jul 08 03:44:39[:error]  ConfigFileParseError: Failed to parse /etc/nova/nova.conf: at /etc/nova/nova.conf:11307, No ':' or '=' found in assignment: 'openstack-config --set*    

so, someone modified /etc/nova/nova.conf, add 2 lines at end:   
openstack-config --set /etc/nova/nova.conf DEFAULT compute_driver libvirt.LibvirtDriver  
openstack-config --set /etc/nova/nova.conf libvirt virt_type kvm  
  
remove these 2 lines, the config is already in the file, leave it as is, they're same as default value. 
 
Now another error   
 *[Thu Jul 12 10:10:16[:error] mod_wsgi (pid=14970): Target WSGI script '/var/www/cgi-bin/nova/nova-placement-api' cannot be loaded as Python module.  
  [Thu Jul 12 10:10:16[:error] mod_wsgi (pid=14970): Exception occurred processing WSGI script '/var/www/cgi-bin/nova/nova-placement-api'.  
......
  [Thu Jul 12 10:10:16[:error] ArgsAlreadyParsedError: arguments already parsed: cannot register CLI option* 

as per https://ask.openstack.org/en/question/6626/nova-arguments-already-parsed-cannot-register-cli-option/  
This error comes up when the attribute has been specified repeatedly and with different values at different config files.  

Restarting httpd service by:  systemctl restart httpd   
now instance could be created succesfully.  

# Manual test placement API #     

Review /var/log/httpd/placement_wsgi_error.log, there is another error message though:     
[Fri Jul 13 00:24:43.697804 2018] [:error] [pid 17714] /usr/lib/python2.7/site-packages/oslo_db/sqlalchemy/enginefacade.py:332: NotSupportedWarning: Configuration option(s) ['use_tpool'] not supported     
[Fri Jul 13 00:24:43.697909 2018] [:error] [pid 17714]   exception.NotSupportedWarning   
  
This error message also shows up when manually testing placement API:    
*$ sudo -H -u nova bash -c '/var/www/cgi-bin/nova/nova-placement-api'    
bash: /var/www/cgi-bin/nova/nova-placement-api: Permission denied   
$ chmod 744 /var/www/cgi-bin/nova/nova-placement-api   
$ sudo -H -u nova bash -c '/var/www/cgi-bin/nova/nova-placement-api'    
/usr/lib/python2.7/site-packages/oslo_db/sqlalchemy/enginefacade.py:332: NotSupportedWarning: Configuration option(s) ['use_tpool'] not supported    
  exception.NotSupportedWarning  *  
STARTING test server nova.api.openstack.placement.wsgi.init_application    
Available at http://localhost.localdomain:8000/  
DANGER! For testing only, do not use in production  
*

Refer to https://github.com/openstack/oslo.db/commit/c432d9e93884d6962592f6d19aaec3f8f66ac3a2  
made following change to eliminate this message:    
*cd /usr/lib/python2.7/site-packages/oslo_db/sqlalchemy    
[root@illinbws111 sqlalchemy(keystone_core)]# diff enginefacade.py enginefacade.py.20180712    
175c175    
                'db_max_retry_interval', 'backend','use_tpool'])  
                'db_max_retry_interval', 'backend'])*






