# Autostart

### Quick links
* [Login Items](#login-items)
* [LaunchD](#launchd)
  * [Launch Agents](#launch-agents)
  * [Autostart Script](#autostart-script)

## Login Items
1. Navigate to `System Settings >General >Login Items`
2. Click the little plus icon and select your app

## LaunchD
`launchd` manages the startup of daemons as needed when traffice comes in on the configured port that 
maps to the service.

* `~/Library/LaunchAgents` are specific to the user and will run when the user is logged in
* `/Library/LaunchAgents` are for all users and will run when any user logs in
* `/Library/LaunchDaemons` are run regarless of users and require `sudo` to load the `.plist`

Loading a job definition does not necessarily mean the job will be started. When the job is started is 
determined by the job definition. When the `RunAtLoad` or `KeepAlive` flags have been specified 
launchd will start the job unconditionally when it has been loaded.

### Launch Agents
When a user logs in a new launchd process will be started for this user. This launchd process will 
scan the agent directories `/System/Library/LaunchAgents /Library/LaunchAgents ~/Library/LaunchAgents`
for job definitions and load them depending on the existence/value of the `Disabled` key

1. Launch a shell
2. Run the oneliner `launchctl unload -w /Library/LaunchAgents/<offending plist>`
2. Example disable Cisco VPN: `launchctl unload -w /Library/LaunchAgents/com.cisco.anyconnect.gui.plist`

### Autostart Script
launchd will scan `/System/Library/LaunchDaemons` and `/Library/LaunchDaemons` for job definitions 
and load them depending on the existence/value of the `Disabled` key

1. Create a `.plist` file according to Apple docs
   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
      <key>Label</key>
      <string>com.user.script</string>
      <key>Program</key>
      <string>/path/to/executable/script.sh</string>
      <key>RunAtLoad</key>
      <true/>
   </dict>
   </plist
   ```
2. Place the `.plist` file in ***/Library/LaunchDaemons***
3. Login to run or manually run with `launchctl load [filename.plist]`
