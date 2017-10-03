***Using CGroups with Yarn***

**What is CGroups?**

CGroups is a mechanism for aggregating/partitioning sets of tasks, and all their future children, into hierarchical groups with specialized behaviour. CGroups is a Linux kernel feature and was merged into kernel version 2.6.24. From a YARN perspective, this allows containers to be limited in their resource usage. A good example of this is CPU usage. Without CGroups, it becomes hard to limit container CPU usage. Currently, CGroups is only used for limiting CPU usage.


Documentation:https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/NodeManagerCgroups.html

**Preparation before configuration:** 

before you go deeper the configuration, you need *sudo* privilege to move on.

1. preparation for *container-executor* file:

        chown root:hyad-all bin/container-executor
        chmod 6050 bin/container-executor
It also require that the privilege of *container-executor.cfg* is needed to be owned by root from root directory to current one. The default path is *${HADOOP_HOME}/etc/hadoop/container-executor.cfg*. If it is somehow not good to change the path of hadoop directory, we can re-compile the *container-executor* file:

Using *cmake -DHADOOP_CONF_DIR=/etc/hadoop* to recompile *container-executor*, so that the path change to */etc/hadoop*

        cd /yuxingch
        tar -zxf hadoop-2.7.4-src.tar.gz
        cd /yuxingch/hadoop-2.7.4-srchadoop-yarn-project/hadoop-yarn/hadoop-yarn-server/hadoop-yarn-server-nodemanager/
        cmake src -DHADOOP_CONF_DIR=/etc/hadoop
        make
        cd targe/usr/local/bin/
        cp container-executor /etc/hadoop/container-executor
Note: *container-executor* file is owned by root and belongs to the same group of your executing users.

2. preparation for *container-executor.cfg* file:

        # because of the extra space left at the end of line of some file format, please delete the comment from original file, otherwise, it may cause the problem of
        # 'Can't get group information for hyad-all(your group name)  - Success.' or  'Can't get configured value for yarn.nodemanager.linux-container-executor.group.'
        yarn.nodemanager.linux-container-executor.group=hyad-all    # first group from my $groups 
        banned.users=root               # your usernames that don't allow to use it
        min.user.id=1000
        allowed.system.users=yuxingch   # your usernames that allow to use it

Check if the configuration is correct or not:
    
        $./bin/container-executor --checksetup
Note: *container-executor.cfg* file is owned by root and belongs to the same group of your executing users from root directory to current directory.
        
**CGroups configuration:** 
       
CGroups is a Linux kernel feature and is exposed via the LinuxContainerExecutor:

    <property>
        <name>yarn.nodemanager.container-executor.class</name>
        <value>org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor</value>
    </property>
This configuration is required for validating the secure access of the container-executor binary:

    <property>
        <name>yarn.nodemanager.linux-container-executor.group</name>
        <value>hadoop</value>
    </property>
Using the LinuxContainerExecutor doesn’t force you to use CGroups. If you wish to use CGroups, the resource-handler-class must be set to CGroupsLCEResourceHandler:

    <property>
        <name>yarn.nodemanager.linux-container-executor.resources-handler.class</name>
        <value>org.apache.hadoop.yarn.server.nodemanager.util.CgroupsLCEResourcesHandler</value>
    </property>

Where the LCE should attempt to mount cgroups if not found:

    <property>
        <name>yarn.nodemanager.linux-container-executor.cgroups.mount-path</name>
        <value>/cgroup</value>
    </property>
    
Below is how I configure this folder:
    
    #drwxrwxrwx  3 root     root
    $mkdir /cgroup 
    #drwxr-xr-x  3 yuxingch hyad-all
    $mkdir /cgroup/cpu

Weather we should limit users or not when nonsecure mode is on:

    <property>
        <name>yarn.nodemanager.linux-container-executor.nonsecure-mode.limit-users</name>
        <value>false</value>
    </property>
Whether the LCE should attempt to mount cgroups if not found - can be true or false:

    <property>
        <name>yarn.nodemanager.linux-container-executor.cgroups.mount</name>
        <value>true</value>
    </property>
Users that allow to user cgroups, separated by comma:

    <property>
        <name>yarn.nodemanager.linux-container-executor.nonsecure-mode.local-user</name>
        <value>yuxingch</value> 
    </property>
**The following settings are related to limiting resource usage of YARN containers:**

CGroups allows cpu usage limits to be hard or soft. When this setting is true, containers cannot use more CPU usage than allocated even if spare CPU is available. This ensures that containers can only use CPU that they were allocated. When set to false, containers can use spare CPU if available. It should be noted that irrespective of whether set to true or false, at no time can the combined CPU usage of all containers exceed the value specified in “yarn.nodemanager.resource.percentage-physical-cpu-limit”:

    <property>
        <name>yarn.nodemanager.linux-container-executor.cgroups.strict-resource-usage</name>
        <value>true</value>
    </property>
This setting lets you limit the cpu usage of all YARN containers. It sets a hard upper limit on the cumulative CPU usage of the containers. For example, if set to 60, the combined CPU usage of all YARN containers will not exceed 60%:

    <property>
     <name>yarn.nodemanager.resource.percentage-physical-cpu-limit</name>
     <value>50</value>
    </property>
