# 脚本

Java服务的监控脚本

```script
#!/bin/bashps -Leo pid,lwp,user,pcpu,pmem,cmd >> /tmp/pthreads.logecho "ps -Leo pid,lwp,user,pcpu,pmem,cmd >> /tmp/pthreads.log" >> /tmp/pthreads.logecho `date` >> /tmp/pthreads.logecho 1pid=`ps aux|grep tomcat|grep cwh|awk -F ' ' '{print $2}'`echo 2echo "pstack $pid >> /tmp/pstack.log" >> /tmp/pstack.log
pstack $pid >> /tmp/pstack.logecho `date` >> /tmp/pstack.logecho 3echo "lsof >> /tmp/sys-o-files.log" >> /tmp/sys-o-files.log
lsof >> /tmp/sys-o-files.logecho `date` >> /tmp/sys-o-files.logecho 4echo "lsof -p $pid >> /tmp/service-o-files.log" >> /tmp/service-o-files.log
lsof -p $pid >> /tmp/service-o-files.logecho `date` >> /tmp/service-o-files.logecho 5echo "jstack -l $pid  >> /tmp/js.log" >> /tmp/js.log
jstack -l -F $pid  >> /tmp/js.logecho `date` >> /tmp/js.logecho 6 echo "free -m >> /tmp/free.log" >> /tmp/free.log
free -m >> /tmp/free.logecho `date` >> /tmp/free.logecho 7echo "vmstat 2 1 >> /tmp/vm.log" >> /tmp/vm.log
vmstat 2 1 >> /tmp/vm.logecho `date` >> /tmp/vm.logecho 8echo "jmap -dump:format=b,file=/tmp/heap.hprof 2743" >> /tmp/jmap.log
jmap -dump:format=b,file=/tmp/heap.hprof >> /tmp/jmap.logecho `date` >> /tmp/jmap.logecho 9echo end
```
