Docs of Apache Storm related projects
==========
[Repository] (https://github.com/ADSC-Cloud/storm)

sync-executors
----------
Keep workers during migration and start/kill executors accordingly.
([git branch] (https://github.com/ADSC-Cloud/storm/tree/sync-executor))  
Implementation: **TODO**

reroute-message
----------
Handle wrongly routed message and reroute them.  
Implementation: **TODO**

1. transfer-local-fn  
Issue: task-id is not local.  
The message may be from an updated/outdated worker.  
Solution: delay and retry (**retry-msg-fn**)

2. transfer-tuples-handler  
Issue: No connection for the message.  
Either (1) the message should be handled locally: the task has become local due to an update;
or (2) a temp connection is needed: the target host is no longer a downstream host due to an update.  
Solution: delay and retry

3. transfer-fn  
Issue: null node+port.  
Either (1) the task has become local right after the local-tasks? check;
or (2) we need a temp connection.  
Solution: Check local-tasks? again for case (1).
If it is not local, schedule to create a temp connection, delay and retry.

4. Lost messages:
	- Outdated worker -> Killed worker:  
	Only detectable at the transport layer. **Won’t handle this case.**
	- Killed executor, but host worker is still alive -> any:  
	Transfer all messages in queues to worker when shutting down the executor. **In progress**
	- Killed worker -> any:  
	<del>
	Keep the worker alive until all messages are sent?  
	What if we need to use the port for a new worker?  
	Resource conflicts: port, log file, heartbeat file, etc.  
	Maybe close the port (possible using receive-thread-shutdown)?
	</del>  
	Supervisor will keep workers alive for 1 sec during shutdown.
