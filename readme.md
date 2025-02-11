Script to backup all my iMessages outside of iCloud. As of writing this, iMessage takes up about 120GB of my 200GB of iCloud. I'm going to switch configuration from `save forever -> delete after 1 year`.

This uses https://github.com/ReagentX/imessage-exporter to export all my chat data to txt files, and then syncs all that data to an ~~S3 bucket~~ Google Drive folder as an archive.

I have this configured to run daily with (LaunchD)[https://www.launchd.info/] configuration. I'm running this on my personal machine which may be close and asleep most of the time, and crontab cannot wake the machine up to run the script. LaunchD _does_ allow for that, so I'm using it.

```zsh
# create the file in LaunchAgents
> touch ~/Library/LaunchAgents/com.pwc.imessage-backup.plist

# load the launch config
> launchctl load ~/Library/LaunchAgents/com.pwc.imessage-backup.plist

# find the status
> launchctl list | grep com.pwc.imessage-backup

# unload it if you need to make a change
> launchctl load ~/Library/LaunchAgents/com.pwc.imessage-backup.plist
```

This script is entirely specific to my setup, but you could easily tweak it for yourself.

## TODO

### Ideas

- Add initial setup script to ease config
- create plist template to be injected with user config options
  - user home directory
  - export path
  - desired export options per imessage-exporter
  - multiple export option schedules (e.g. text daily, html weekly, full monthly)
- Check whether plist exists before creating it
- Add option to run on-demand
- Check for rsync, imessage-exporter, and other dependencies
- Use a click cookiecutter to make this a proper CLI tool
- grab a razor and see if I can shave my way out of this herd of yak

These should help me fix up my current setup, stored in iTerm2 snippets to be run as one-liners:

### Full HTML with attachments

```zsh
# full imessage-exporter run with attachments, undated
# tmp directory required to avoid some weird bug with the exporter
cd /Volumes/Acasis\ TBU405PROM1\ 4TB\ WD_BLACK\ SN850X/imessage_export/full/ && \
imessage-exporter \
--export-path /Volumes/Acasis\ TBU405PROM1\ 4TB\ WD_BLACK\ SN850X/imessage_export/full/tmp \
--format html \
--no-lazy \
--copy-method clone && \
rsync -avhP \
--remove-source-files \
'/Volumes/Acasis TBU405PROM1 4TB WD_BLACK SN850X/imessage_export/full/tmp/' \
'/Volumes/Acasis TBU405PROM1 4TB WD_BLACK SN850X/imessage_export/full/' && \
rsync -avhP \
'/Volumes/Acasis TBU405PROM1 4TB WD_BLACK SN850X/imessage_export/' \
root@archive.local:/mnt/user/iMazing/imessage_export && \
imessage-exporter -d
```

### HTML only

```zsh
# html imessage-exporter run, no attachments
# need to fix date rotation, also the inline date doesn't pass through to later commands
cd /Volumes/Acasis\ TBU405PROM1\ 4TB\ WD_BLACK\ SN850X/imessage_export/html/ && \
imessage-exporter \
--export-path /Volumes/Acasis\ TBU405PROM1\ 4TB\ WD_BLACK\ SN850X/imessage_export/html/$(date '+%Y%m%d_%H%M%S')/ \
--format html \
--no-lazy &&  \
rsync -avhP \
'/Volumes/Acasis TBU405PROM1 4TB WD_BLACK SN850X/imessage_export/' \
root@archive.local:/mnt/user/iMazing/imessage_export && \
imessage-exporter -d
```

### Text only

```zsh
# txt imessage-exporter run, no attachments
cd /Volumes/Acasis\ TBU405PROM1\ 4TB\ WD_BLACK\ SN850X/imessage_export/txt/ && \
imessage-exporter \
--export-path /Volumes/Acasis\ TBU405PROM1\ 4TB\ WD_BLACK\ SN850X/imessage_export/txt/$(date '+%Y%m%d_%H%M%S')/ \
--format txt \
--no-lazy &&  \
rsync -avhP \
'/Volumes/Acasis TBU405PROM1 4TB WD_BLACK SN850X/imessage_export/' \
root@archive.local:/mnt/user/iMazing/imessage_export && \
imessage-exporter -d
```
