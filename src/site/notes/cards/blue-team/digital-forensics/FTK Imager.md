---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/ftk-imager/"}
---

[[map-of-contents/blue-team\|blue-team]]
### Introduction
---
A forensic image that can copy bit by bit of a storage device as well as producing a hash for integrity.

- Some useful introduction notes [here.](obsidian://open?vault=notes&file=Atlas%2FCyber%20Defense%2F2023-12-09)
## Demo
---
Start Menu -> Disk Management:

![FTK Imager.png](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager.png)
Default settings should be good and then initialize the newly created partition: ![FTK Imager-1.png](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager-1.png)
Then allocate the disk: (choose: new simple volume then default should be good) ![FTK Imager-2.png](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager-2.png)
### Creating Disk Image
---
**File -> Create Disk Image**: 

![FTK Imager-3.png|350](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager-3.png)
Ideally the **physical drive** option should be selected as it captures everything as for **logical drive**, it is only useful for partitions and might not capture the whole picture, **image files** is self-explanatory, a copy of a copy, **Contents of a Folder**, in some legal cases copying the full drive is not allowed.

- `EO1` - disk image type should be selected for compatibility with other forensic tools and advance features.

![FTK Imager-4.png|450](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager-4.png)
It is important to add physical description about the hard disk or device that we are taking an image from then saving the image should be proper, or well organized and descriptive.

![FTK Imager-5.png](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager-5.png)
Then:

![FTK Imager-6.png](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager-6.png)
### Collecting Volatile Data
---
**File -> Capture Memory**

Ideally the filename should be device descriptive including the hostname:
![FTK Imager-7.png|300](/img/user/cards/blue-team/digital-forensics/images/FTK%20Imager-7.png)
- **Pagefile** is for extended memory, capture extended memory these are disk storage space that is used for extending the RAM. 

After creation of the `.mem` file then **calculate the hash**, as calculating the hash before the image taken is not ideal as it constantly changes then **document** the following.


**Organize and refine notes after each session including adding meta-tags**

