I would like to make a directory of everyone's devices out there so we can see any differences in the hardware of the different models and bios revisions.

We have noticed that some BIOSes are a version 6 and otehr are a version 7 and that can affect the success rate of flashing a modified bios

Plans are in the works to get CoreBoot or another open bios functional, but hardware documentation is necessary.

/
/

There are two Flash chips located on the board.
    A 25LB64C - which is 8MBs in size and holds the EFI BIOS
    A 25Q80   - which is 1MB  in size and holds what we think is a backup of the 128KB internal flash to the ITE IO chip it is next to.

/
/

Upload Rules
    Make a pull request and put the following into the dumped directory
    Only request uploads to the dumped folder, modified are for us to upload changes.


Put Original Dumped bios into the "Dumped" folder
Put modified bios files in "modified"
    Create seperate folders for the V6 and V7 Bios in your archive
    Zip the Bioses together with one folder containing the 28Q80 for the ITE Chip and another containing Device Bios
    Please use '.zip', '.7z', or '.tar.gz' files when uploading archive

    Inside of the "model_info.txt" file please put the following
        Your name/handle
        Serial Number
        Whether it is a wifi or non-wifi model
        Any additional information you'd like to share regarding your device.
Example of rules
    name-date-#.zip
    ├── model_info.txt
    ├── 25LB64C - BIOS FLASH
    │   └── 25LB64C-name.bin
    └── 25Q80 - ITE FLASH
        └── 25Q80-name.bin
i.e
sark-030724-1.zip
├── model_info.txt
├── 25LB64C - BIOS FLASH
│   └── 25LB64C-sark.bin
└── 25Q80 - ITE FLASH
    └── 25Q80-sark.bin

Modified Bioses should follow these rules as well to maintain consitencey and organization

Any releases will be dated in the release folder.
