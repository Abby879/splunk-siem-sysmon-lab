Created a Splunk search to identify Sysmon EventCode 1 process creation events from the monitored Windows endpoint.

Created a Sysmon-based Splunk search to identify PowerShell execution events on the monitored Windows endpoint. The event details confirmed the executable path, command line, user context, and parent process.

Created a Splunk detection search for failed Windows login attempts using Security EventCode 4625 and validated it by generating multiple incorrect password attempts on the monitored endpoint.

Built a Splunk dashboard to visualize failed login attempts, PowerShell execution, and Sysmon process creation activity from the monitored Windows endpoint.