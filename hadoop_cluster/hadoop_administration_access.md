**Hadoop Administration** 

**HDFS and MapReduce Job Monitoring **
HDFS administration includes monitoring the HDFS file structure, locations, and the updated files.
MapReduce administration includes monitoring the list of applications, the configuration of nodes, application status, etc.

    URL: "http://ukkoserver:50070
    
    e.g http://ukko144.hpc.cs.helsinki.fi:50070

This URL is not accessible from the internet as Ukko servers are only accessible from University's network. To browse the Administration page, create ssh session with option -X or -Y (check the manual page for detail). This option enables to use a graphical client in untrusted and trusted environment respectively. 

**Access from Outside**.
First connect to the interactive servers(melkki, melkinpassi or melkinkari) via secure shell (ssh) with additional option i.e.  -X or -Y.
e.g

    ssh -X  user@melkinpassi.cs.helsinki.fi
    Enter Password

After logging on melkinpassi, you can access the Administration page with per-installed chromium browser on the server.

    $chromium-browser http://hadoop_ukkoserver:50070
    
    e.g $chromium-browser http://ukko144.hpc.cs.helsinki.fi:50070

In the same fashion, you can access the MapReduce  Job Monitoring page.

    $chromium-browser http://ukkoserver:8088
    e.g http://ukko144.hpc.cs.helsinki.fi:8088
    


