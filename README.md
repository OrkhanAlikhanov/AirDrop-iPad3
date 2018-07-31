# AirDrop for iPad 3

This is an **attempt** to port AirDrop capability to iPad 3 which was once reported to be successfully working in iOS 7.

All files (except in DEBIAN folder) is copied from other iOS firmware (iPod5,1_9.3.5_13G36). Those are <sup>[\[1\]][1]</sup>:
- /System/Library/AccessibilityBundles/SharingFramework.axbundle
- /System/Library/Audio/UISounds/Modern/airdrop_invite.caf
- /System/Library/LaunchDaemons/com.apple.sharingd.plist
- /System/Library/PrivateFrameworks/Sharing.framework
- /System/Library/SpringBoardPlugins/Sharing.servicebundle
- /usr/libexec/sharingd


## Why iPod 5 files?
I copied files from iPod 5 since CPU type of iPad 4 (iPad3,4_9.3.5_13G36) was not compatible with iPad 3. Running `/usr/libexec/sharingd` from iPad 4 complaing about cpu type.
```bash
> /usr/libexec/sharingd
> /usr/libexec/sharingd: Bad CPU type in executable
```
But it seems to be working with the file from iPod 5



## About LaunchDaemons

Note that `com.apple.sharingd.plist` needs to be copied to `Library/LaunchDaemons/` because starting iOS 8, `launchctl` load/unload no longer works with daemons in `/System/Library/LaunchDaemons/`, it exits with an error <sup>[\[2\]][2]</sup>: 
```bash
> launchctl load /System/Library/LaunchDaemons/com.apple.sharingd.plist
> /System/Library/LaunchDaemons/com.apple.sharingd.plist: The specified service path was not in the service cache
```
But it can successfully load/unload daemons from `/Library/LaunchDaemons/`, so `com.apple.sharingd.plist` is put there.

### Last thing
Let's try executing `/usr/libexec/sharingd`

```bash
> /usr/libexec/sharingd
> dyld: Library not loaded: /System/Library/PrivateFrameworks/Sharing.framework/Sharing
  Referenced from: /usr/libexec/sharingd
  Reason: image not found
Trace/BPT trap: 5

```

This is because since iOS 3.1, all default (private and public) libraries have been combined into a big cache file in `/System/Library/Caches/com.apple.dyld/dyld_shared_cache_armX` (where X can be v6, v7, v7s or 64) to improve performance. <sup>[\[3\]][3]</sup>

Those private frameworks were being shipped with iPhone SDK until Xcode 7.3 (or maybe iOS 8 SDKs) removed them all. So the only way to get `/System/Library/PrivateFrameworks/Sharing.framework/Sharing` library is to take it out of `dyld_shared_cache_arm7`.

iPhone Dev Wiki lists] multiple [tools][4] for extracting the `dyld_shared_cache`. However, none of the works for iOS 9. There is a [great post][5] by [@zhuowei](https://github.com/zhuowei) analyzing all of them and trying to create [a working extractor](https://github.com/zhuowei/dsc_extractor_badly). Hope he will succeed!


## How to create a deb file
After getting `Sharing` dynamic library from `dyld_shared_cache_arm7` we can create a deb file.
```bash
git clone https://github.com/OrkhanAlikhanov/AirDrop-iPad3.git AirDrop
find AirDrop -name ".DS_Store" -delete # remove all .DS_Store files
dpkg-deb -Zgzip -b AirDrop # will create AirDrop.deb
```


[1]: http://theiphonewiki.com/wiki/AirDrop
[2]: http://iphonedevwiki.net/index.php/Updating_extensions_for_iOS_8#Daemons
[3]: http://theiphonewiki.com/wiki//System/Library/Frameworks
[4]: http://iphonedevwiki.net/index.php/Dyld_shared_cache#Cache_extraction
[5]: http://worthdoingbadly.com/dscextract/