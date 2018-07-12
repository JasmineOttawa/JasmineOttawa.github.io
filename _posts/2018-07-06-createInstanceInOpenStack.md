---
layout: post
title: "create instance in OpenStack Queens got - 500 Internal Server Error"
date: 2018-07-06
---

Q -- create instance on openstack queens got error "500 Internal Server Error"  
openstack server create --image Ubuntu_16.04_LTS --flavor ... --key-name ... --nic net-id=.... ttt  
openstack server list  
  
investigation --   
/var/log/nova/nova-scheduler.log shows 500 internel server error :   
  *The server encountered an internal error or misconfiguration and was unable to complete your request. 
  Please contact the server administrator at [no address given] to inform them of the time this error occurred,  and the actions you performed just before this error.    
   More information about this error may be available in the server error log.*      

go to server error log, /var/log/httpd/placement_wsgi_error.log, First error shows at Jul 08:   
 *[Sun Jul 08 03:44:39[:error]  mod_wsgi (pid=14961): Target WSGI script '/var/www/cgi-bin/nova/nova-placement-api' cannot be loaded as Python module.  
  [Sun Jul 08 03:44:39[:error]  mod_wsgi (pid=14961): Exception occurred processing WSGI script '/var/www/cgi-bin/nova/nova-placement-api'.  
  [Sun Jul 08 03:44:39[:error]  Traceback (most recent call last):  
  [Sun Jul 08 03:44:39[:error]    File "/var/www/cgi-bin/nova/nova-placement-api", line 54, in <module>  
  [Sun Jul 08 03:44:39[:error]      application = init_application()  
  [Sun Jul 08 03:44:39[:error]    File "/usr/lib/python2.7/site-packages/nova/api/openstack/placement/wsgi.py", line 54, in init_application  
  [Sun Jul 08 03:44:39[:error]      config.parse_args([], default_config_files=[conffile])  
  [Sun Jul 08 03:44:39[:error]    File "/usr/lib/python2.7/site-packages/nova/config.py", line 52, in parse_args  
  [Sun Jul 08 03:44:39[:error]      default_config_files=default_config_files)  
  [Sun Jul 08 03:44:39[:error]    File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 2502, in __call__  
  [Sun Jul 08 03:44:39[:error]      else sys.argv[1:])  
  [Sun Jul 08 03:44:39[:error]    File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 3166, in _parse_cli_opts  
  [Sun Jul 08 03:44:39[:error]      return self._parse_config_files()  
  [Sun Jul 08 03:44:39[:error]    File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 3183, in _parse_config_files  
  [Sun Jul 08 03:44:39[:error]      ConfigParser._parse_file(config_file, namespace)  
  [Sun Jul 08 03:44:39[:error]    File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 1950, in _parse_file  
  [Sun Jul 08 03:44:39[:error]      raise ConfigFileParseError(pe.filename, str(pe))  
  [Sun Jul 08 03:44:39[:error]  ConfigFileParseError: Failed to parse /etc/nova/nova.conf: at /etc/nova/nova.conf:11307, No ':' or '=' found in assignment: 'openstack-config --set*    

so, someone modified /etc/nova/nova.conf, add 2 lines at end:   
openstack-config --set /etc/nova/nova.conf DEFAULT compute_driver libvirt.LibvirtDriver  
openstack-config --set /etc/nova/nova.conf libvirt virt_type kvm  
  
remove these 2 lines, the config is already in the file, uncomment them in the corresponding section  
 
Now got another error   
 *[Thu Jul 12 10:10:16[:error] mod_wsgi (pid=14970): Target WSGI script '/var/www/cgi-bin/nova/nova-placement-api' cannot be loaded as Python module.  
  [Thu Jul 12 10:10:16[:error] mod_wsgi (pid=14970): Exception occurred processing WSGI script '/var/www/cgi-bin/nova/nova-placement-api'.  
  [Thu Jul 12 10:10:16[:error] Traceback (most recent call last):  
  [Thu Jul 12 10:10:16[:error]   File "/var/www/cgi-bin/nova/nova-placement-api", line 54, in module  
  [Thu Jul 12 10:10:16[:error]     application = init_application()  
  [Thu Jul 12 10:10:16[:error]   File "/usr/lib/python2.7/site-packages/nova/api/openstack/placement/wsgi.py", line 54, in init_application  
  [Thu Jul 12 10:10:16[:error]     config.parse_args([], default_config_files=[conffile])  
  [Thu Jul 12 10:10:16[:error]   File "/usr/lib/python2.7/site-packages/nova/config.py", line 35, in parse_args  
  [Thu Jul 12 10:10:16[:error]     log.register_options(CONF)  
  [Thu Jul 12 10:10:16[:error]   File "/usr/lib/python2.7/site-packages/oslo_log/log.py", line 250, in register_options  
  [Thu Jul 12 10:10:16[:error]     conf.register_cli_opts(_options.common_cli_opts)  
  [Thu Jul 12 10:10:16[:error]   File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 2440, in inner  
  [Thu Jul 12 10:10:16[:error]     result = f(self, *args, kwargs)  
  [Thu Jul 12 10:10:16[:error]   File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 2662, in register_cli_opts  
  [Thu Jul 12 10:10:16[:error]     self.register_cli_opt(opt, group, clear_cache=False)  
  [Thu Jul 12 10:10:16[:error]   File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 2444, in inner  
  [Thu Jul 12 10:10:16[:error]     return f(self, *args, kwargs)  
  [Thu Jul 12 10:10:16[:error]   File "/usr/lib/python2.7/site-packages/oslo_config/cfg.py", line 2654, in register_cli_opt  
  [Thu Jul 12 10:10:16[:error]     raise ArgsAlreadyParsedError("cannot register CLI option")  
  [Thu Jul 12 10:10:16[:error] ArgsAlreadyParsedError: arguments already parsed: cannot register CLI option* 

as per https://ask.openstack.org/en/question/6626/nova-arguments-already-parsed-cannot-register-cli-option/  
This error comes up when the attribute has been specified repeatedly and with different values at different config files.  
  
so there's other changes in nova.conf, there's no backup left from prior change, how to restore nova.conf?   


