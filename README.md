# Service Manager (sm)

Management user service like systemd.

## Tech Notes
 * use ```$ExecStarCommand > ${SM_DIR}/log/${SERVICE_NAME}.log  2>&1 &``` command to execute user defined command, stdout and stderr redirected to log file.
 * use SERVICE_NAME to management service instead of type long command line,
    the command line saved in ${SM_DIR}/user/*.service file, such as foo.service, SERVICE_NAME is "foo".
 * use pid to monitor the status of service,
    use kill -9 <pid> to stop the service.

## Install and example
 * (1) copy "sm" command to /usr/bin
 * (2) "sm init" to create ${HOME}/sm directory and sub directory.
 * (3) put some service file to ${HOME}/sm/user directory, such as:
```
cat > ${HOME}/sm/user/foo.service <<EOF
[Service]
ExecStart=/usr/bin/tail -f /var/log/system.log
EOF
```
 * (4) "sm enable foo" to enable foo service.
 * (5) "sm run foo" to run foo service.
> start foo ...
> /usr/bin/tail -f /var/log/system.log
> started, pid = 14648.
 * (6) "sm status foo" to show status of foo service.
> active (running)  [foo] -- 14648 /usr/bin/tail -f /var/log/system.log
 * (7) "sm kill foo" to stop foo service.
> stop foo (pid=14648) ...

## Usage
```
usage: sm <r|run|start>|<k|kill|stop>|<re|restart> <service-name>
       sm <run-all>|<kill-all>
       sm <s|status> [<service-name>]
       sm <e|enable>|<d|disable> <service-name>
       sm <l|list>
       sm exec <service-name>  # forgrand run service without in ${SM_DIR}/user directory
       sm <init>
```
## Manual
  `$SM_DIR` default is `$HOME/sm`， type `sm` without arguments will show it.
```  
  ${SM_DIR}/user/ -- *.service files, defined by user
  ${SM_DIR}/work/ -- symbol link of *.service, it created or deleted by <enable> and <disable> command,
                    prohibit manually work in this dir for will be removed automaticlly.
  ${SM_DIR}/pid/ -- *.pid files, auto generated
  ${SM_DIR}/log/ -- *.log files, auto generated
  service file format:
    [Service]
    ExecStart=<command> [arguments]  -- ONLY ABSOLUTE PATH SUPPORTED.
    WorkingDirectory=/home/user/sm -- OPTIONAL, default is ${SM_DIR} when not set
    Restart=never -- OPTIONAL, default is never, set always will be restart when sm-monitor is running  
  Auto Restart service when crash: 
  (1) sm enable sm-monitor  # sm init will generate $HOME/sm/user/sm-monitor.service and enable it
  (2) sm run sm-monitor
  (3) add Restart=always to foo.service
  if foo crash, sm-monitor while restart it, checked every 5 minutes.
  don't worry about sm kill <service-name>, it will not be restart automaticlly.  

  Use systemd monitor the serivce:
    /usr/bin/sm init
    sudo cp ~/sm/user/sm-monitor.service /lib/systemd/system/
    sudo systemctl daemon-reload
    sudo systemctl enable sm-monitor
    sudo systemctl start sm-monitor
    sudo systemctl status sm-monitor

  Content of sm-monitor.service:
    [Service]
    ExecStart=/usr/bin/sm monitor
    Restart=always
    User={username}

    [Install]
    WantedBy=default.target
```
## Service Group Management
  enable a service prefix with a sub-dir, it will create symbol link in sub-dir of work directory.
  user directory not support sub-dir. it can be run-all or kill-all independent, example: 
* (1) sm enable test/foo
> enable /Users/renren/sm/user/foo.service to /Users/renren/sm/work/test/foo.service
* (2) sm enable test/bar
* (3) sm run-all test
>  start all service in test sub directory
* (4) sm kill-all nosub
> will stop-all service in work directory without sub-directory.
```
  usage(with more description): sm <r|run|start>|<k|kill|stop>|<re|restart> [work-subdir-name/]<service-name>
       sm <run-all|start-all>|<kill-all|stop-app> [<work-subdir-name>|nosub]
       sm <s|status> [[work-subdir-name/]<service-name>|<work-subdir-name>|nosub]
       sm <e|enable>|<d|disable> [work-subdir-name/]<service-name>
```
