# Using Cyberduck on LUMI

Using Cyberduck requires to first create a profile for LUMI-O. Put the following
in the file `lumio.cyberduckprofile`.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Protocol</key>
        <string>s3</string>
        <key>Vendor</key>
    <string>lumio</string>
        <key>Scheme</key>
        <string>https</string>
        <key>Description</key>
        <string>LUMI-O storage</string>
        <key>Port Configurable</key>
        <false/>
        <key>Default port</key>
    <string>443</string>
        <key>Username Configurable</key>
        <true/>
    <key>Hostname Configurable</key>
    <false/>
    <key>Default Hostname</key>
    <string>lumidata.eu</string>
        <key>Properties</key>
        <array>
            <string>s3.bucket.virtualhost.disable=true</string>
            <string>s3.versioning.enable=false</string>
        <string>s3.listing.versioning.enable=false</string>
        <string>b2.listing.versioning.enable=false</string>
        <string>b2.listing.versioning.enable=false</string>
        <string>versioning.enable =false</string>
        <string>versioning.move.enable=false</string>
        <string>versioning.enable =false</string>
        <string>versioning.delete.enable=false</string>
        <string>versioning.enable=false</string>
        <string>s3.versioning.enable=false</string>
        <string>googlestorage.listing.versioning.enable=false</string>
        <string>s3.listing.versioning.enable=false</string>
        <string>s3.versioning.enable=false</string>
        <string>s3.listing.versioning.enable=false</string>
        <string>s3.listing.versioning.enable=false</string>
        </array>
    </dict>
</plist>
```

[Download here](lumio.cyberduckprofile)

Installing the profile on Cyberduck for MacOS is easy: simply execute

```
open lumio.cyberduckprofile
```

in a console window (or double-click on the file in Finder).
