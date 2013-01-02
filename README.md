gimble
======
Version: v.0.1.0
----------------

A micro logging utility for Windows PowerShell with unique logfile names and auto-purge by date and/or file-count.

### gimble: "To make holes as does a gimlet." - Lewis Carrol 

Usage:
------

This script must be included (dot-sourced, etc) in your script
in which you'd like to use the two logging functions. 
When your script runs, gimble will automatically create a new 
logfile specific to that execution, and dynamically log
whatever your script tells it to.

Include Examples
----------------

    .\$ps_ExecPath\powershell_functions\gimble_scriptlogger.ps1 $ps_LogPath
    .\util\gimble_scriptlogger.ps1 -path 'C:\Some Path\ToMy\LogFiles'
    .\gimble_scriptlogger.ps1 'C:\Some Path\ToMy\LogFiles'

Logfiles
--------

Each script execution will create a log file in a directory 
which you specify, with a filename like this: 

    MyScriptIdentifier_MyMachineName_2012-09-10_0145-48pm.txt

(if you've specified a script identifier)
or

    MyMachineName_2012-09-10_0145-48pm.txt

(if you have not specified a script identifier).

Logfile filename Examples:
--------------------------

    magus_2012-09-10_0145-48pm.txt (no identifier)
    portquery_NDPRRBACKUP_2012-09-10_0200-41pm.txt (identifier, 'portquery')

Logfile variables:
------------------

The active gimble logfile is accessible in parent (calling) scripts
by name and by fully qualified path and name, as:

    $ps_gimble_Logfile
    $ps_gimble_QualifiedLogfile

I tried to make these variable names reasonably unique.


Provided Functions
------------------

When included in another powershell script, 
gimble provides two simple functions:

    LogAndOutput ($textBlock)
    LogOnly ($textBlock)    

Any single-line or multi-line text that you pass to
LogAndOutput() will be written to the log, realtime,
as the script executes, and will also be written to
the console or standard out.
     
Any single-line or multi-line text that you pass to
LogOnly() will be written to the log, realtime,
as the script executes, but will not be written to
the console or standard out.


HELP PARAMETERS
---------------

    gimble_scriptlogger.ps1 -U[u]sage | -G[g]et-H[h]elp | -H[h]elp | U[u]usage | G[g]et-H[h]elp | H[h]elp 

REQUIRED PARAMETER
------------------

There is one required parameter: The path where you want your log files placed.

	-path    Fully qualified path/directory for Logfile output.
  
ALL PARAMETERS
--------------

    -path       [String] Fully qualified path/directory for Logfile output.

    -ident      [String] Identifier, Log file prefix: 
                    Identifier_MachineName_YYYY-MM-DD_HHMM-SSpm.txt
                    Used to identify a log-file as generated by a particular script
                    if you implement a shared logging directory.

    -maxdays    [Integer] logfile purge value, can be used along with -maxfiles.
                    Used to prevent your log file directory from filling up.
                    Maximum age of logfiles to keep in log directory, which have
                    the active Identifier in their name.
                The -purgeglobal flag will change this behavior, to be
                    maximum age of any logfiles to keep in log directory.
                [no value] = Do not perform any file purges by logfile count
                0 = Do not perform any file purges by logfile count

    -maxfiles   [Integer] logfile purge value, can be used along with -maxdays.
    	            Used to prevent your log file directory from filling up.
                    Maximum files to keep in log directory, which have
                    the active Identifier in their name.
                The -purgeglobal flag will change this behavior, to be
	                maximum files to keep in log directory, with any filename.
                [no value] = Do not perform any file purges by logfile count
                0 = Do not perform any file purges by logfile count

More Include Examples (Quick Examples of Purge Settings)
--------------------------------------------------------

   	.\gimble_scriptlogger.ps1 -path $ps_LogPath -ident 'bootstrapbldr' -maxdays 1 -purgeglobal

= Keep only today's script logs in the log folder (and, these logfiles have prefix 'bootstrapbldr').

    .\gimble_scriptlogger.ps1 $ps_LogPath 'bootstrapbldr' 1 0 -purgeglobal

= Same as above.

    .\gimble_scriptlogger.ps1 $ps_LogPath -maxfiles 50

= Keep a maximum of 50 text files in the log folder (and log files have no prefixes).

    .\gimble_scriptlogger.ps1 $ps_LogPath 'site-pings' -maxdays 30

= Keep text files with the filename prefix 'site-pings' in the log folder for 30 days.

    .\gimble_scriptlogger.ps1 $ps_LogPath 'site-pings' -maxfiles 10

= Keep a maximum of 10 text files with prefix 'site-pings' in the log folder.

    .\$ps_CurrentPath\powershell_functions\gimble_scriptlogger.ps1 $ps_LogPath 'wabe' -maxfiles 10 -purgeglobal

= Keep a maximum of 10 text files in the log folder (and, new logfiles have prefix 'wabe').


USAGE EXAMPLE
-------------
    
    LogAndOutput 'Copying files to development deploy...'
    try {
        foreach ($filename in $files) {
            LogAndOutput ('  --> Copying ' + $filename)
            Copy-Item $repo\currentpath\img\$filename $deploypath\img -force | Out-Null
        }
        LogAndOutput 'Copied files to development deploy.'
    }
    catch {
        LogAndOutput 'Error: Can not copy files to deploy location.'
        LogOnly ($error[0].Exception.Message)
        LogAndOutput ('See the logfile at ' + $ps_QualifiedLogfile + ' for more information.')
        LogAndOutput 'Execution halting.'
        Exit
    }
