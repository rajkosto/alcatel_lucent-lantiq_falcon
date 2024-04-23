To flash these firmwares on your G-010S-P stick, you must first upload them via SCP to the stick's /tmp folder like so:

`scp -O -oKexAlgorithms=+diffie-hellman-group1-sha1 -oHostKeyAlgorithms=+ssh-dss firmware.img ONTUSER@192.168.1.10:/tmp/firmware.img`

If you are on Windows, you can use WinSCP to drag the file to the /tmp folder (just make sure you set the "File protocol:" combobox at the top to SCP instead of SFTP or you will get an error that the SFTP server is missing),
or you can use a commandline utility from PuTTY like so:

`pscp.exe -scp -oKexAlgorithms=+diffie-hellman-group1-sha1 -oHostKeyAlgorithms=+ssh-dss firmware.img ONTUSER@192.168.1.10:/tmp/firmware.img`

Then verify the md5sum of the file on the stick matches what you uploaded with `md5sum /tmp/firmware.img`

After that, determine if you are currently booted into image0, or image1 on the stick, either with:

`fw_printenv committed_image`, if that's 0 then you are on image0, and if that's 1 then you are on image1

Or, if you do `cat /proc/mtd | grep image`, if that says image0, you are on image1, and if that says image1, then you are on image0

Depending on which firmware image you are on, only one of the following steps will work:

If you are on image0 (flashing image1):
```
mtd -e image1 write /tmp/firmware.img image1
fw_setenv image1_version PUTVERSIONFROMNAMEOFIMGHERE
fw_setenv image1_is_valid 1
fw_setenv target oem-generic
fw_setenv committed_image 1
```

If you are on image1 (flashing image0):
```
mtd -e image0 write /tmp/firmware.img image0
fw_setenv image0_version PUTVERSIONFROMNAMEOFIMGHERE
fw_setenv image0_is_valid 1
fw_setenv target oem-generic
fw_setenv committed_image 0
```

Replace `PUTVERSIONFROMNAMEOFIMGHERE` in those commands with the name of the file you uploaded in the first step (without the .img suffix)

If you pick the wrong set of commands to run, the first one will fail, do not proceed further if that is the case !

After this, reboot the stick to boot the new firmware, and optionally do the same set of steps but for the other image
