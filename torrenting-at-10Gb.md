10Gb torrenting
===============

On switching to a 10Gb internet-over-fiber connection, I found my torrenting speeds dropped to almost nothing - I'd get maybe 40Kb for a few seconds and then it would stop, only to resume a few seconds later.

The primary issue seems to have been writing to a networked NAS - the relevant buffers kept filling up and blocking things.

So the main solution is simply to always save downloading torrents to a local SSD and only afterwards move them to the NAS.

On the advise of Gemini, I also made a number of changes it said where more appropriate for very high speed connection:

```
# Increase the memory cache to 512MB to buffer 10Gb bursts
$ defaults write org.m0k.transmission CacheSize -int 512

# Disable uTP (Micro Transport Protocol) to stop the "bursty" throttling
# In the plist, this is often 'UTPEnabled'
$ defaults write org.m0k.transmission UTPEnabled -bool false

# Limit global connections to prevent the CPU from choking on overhead
$ defaults write org.m0k.transmission PeeredLimitGlobal -int 300

# Ensure the app allocates disk space immediately to prevent fragmentation
$ defaults write org.m0k.transmission Preallocation -int 1
```

Note: `defaults write org.m0k.transmission` is macOS specific, on other platforms there are similar settings in the Transmission `settings.json` file.

Making these changes wasn't the crucial change, moving from the NAS to the local SSD reduced the transfer time from about a day to under a minute.
