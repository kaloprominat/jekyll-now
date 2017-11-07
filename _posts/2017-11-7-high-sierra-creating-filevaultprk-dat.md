---
layout: post
title: Creating macOS High Sierra 10.13 FileVault individual recovery key escrow payload /var/db/FileVaultPRK.dat manually
---

MacOS High Sierra introduced a brand new [Apple File System](https://en.wikipedia.org/wiki/Apple_File_System "Apple File System"), which supports native full disk encryption known as FileVault. If you are using some kind of enterprise mobililty management system, like [Jamf Pro](https://www.jamf.com/products/jamf-pro/), you may be familiar with Apple's macOS FileVault [recovery key redirect feature](https://mosen.github.io/profiledocs/payloads/fv2redirect.html). It allows you to force managed macs to redirect individual FileVault recovery keys to your management server. Until high sierra released, recovery key was being sent just in time, when FileVault is enabled, or when new individual recovery key is issued. If you loose this key for some reasons, you will not be able to retreive it since it isn't kept anywhere else, including managed mac. Until high sierra, to keep mac's recovery keys in valid state, we have to trigger key reissue for new key to be redirected. It always was tricky and worked not very well. High sierra, regardless of all bugs and glitches, brought to us a new way of [FileVault individual key escrow](https://mosen.github.io/profiledocs/payloads/fvescrow.html). For now, with high sierra, we are able to force managed mac to store its own FileVault individual recovery key, encrypted with specified certificate from your management server. Thanks to PKI, the actual key is not allowed to be decrypted by anyone, except you management server. It's located at path **/var/db/FileVaultPRK.dat**. It makes escrow scheme more relible, as for now recovery key can be collected just like inventory regulary. The bad thing comes up, when we're trying to update macOS Sierra 10.12 with enabled FileVault to macOS High Sierra 10.13. While updating, JHFS+ FileVault is being converted to native APFS full disk encryption. And after that, while individual and institutional recovery keys stays valid and can be used to unlock and decrypt FileVault encrypted drive, FileVaultPRK.dat isn't being created. This may cause situation, when your management server is missing recovery key or stores obsolete one. Fortunately, there is a way to create escrow payload FileVaultPRK.dat and store it in it's place prior the update to high sierra. You just need to ensure, that your management server is capable of new high sierra escrow feature and its escrow encryption certificate is stored on managed mac. In case if you are using Jamf Pro, it should be capable since version 9.101 and should have own FileVault escrow encryption certificate with **CN=JSS Built-In Signing Certificate** and **OU=FILEVAULT2COMM**. To create encrypted escrow FileVaultPRK.dat, just place somewhere plain text file, containing recovery key and run in terminal:
>$ security cms -E -i /tmp/raw_key.txt -o /var/db/FileVaultPRK.dat -r "JSS Built-In Signing Certificate" -v
>received commands
Got default certdb
cmsg [0x7fae9501a020]
arena [0x7fae93716510]
password [NULL]
input len [29]
47 34 44 51 2d 59 39 41 52 2d 51 4d 44 5a 2d 4d 55 54 51 2d 4c 51 4c 46 2d 38 57 36 44 encoding passed
wrote to file

After that you will get prepared recovery key escrow payload at /var/db/FileVaultPRK.dat, which will stay there after update to High Sierra and could be collected and decrypted later by your management server.

The difficult part here is to deliver actual plain recovery key from your management server to managed mac.