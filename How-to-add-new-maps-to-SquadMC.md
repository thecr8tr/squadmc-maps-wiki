Adding new maps to SquadMC is quite involved. As this is a web app with the goal of supporting mobile devices such as smartphones, we have to add a few optimization steps in order to get the performance where we want it to be. This includes splitting the image into smaller tiles and optimizing the heightmap.

The following steps will guide you to add the map "Lashkar Valley" to SquadMC.

## Contents

* [Export minimap & heightmap from Squad SDK](#export-minimap--heightmap-from-squad-sdk)
* create minimap layer
* create heightmap.jpg
* create heightmap layer

## Export minimap & heightmap from Squad SDK

1. Open Squad SDK
   * it will be needed for a later step

2. export minimap

![image](https://user-images.githubusercontent.com/9431420/90961022-21bd9c80-e4a6-11ea-82da-935d969a5f94.png)

3. load AAS or RAAS layer of target map

![image](https://user-images.githubusercontent.com/9431420/90961057-73662700-e4a6-11ea-9375-ee5b422fe7f9.png)

4. export heightmap

![image](https://user-images.githubusercontent.com/9431420/90961104-ccce5600-e4a6-11ea-9da4-efba4f9f8f47.png)

5. write down landscape scale
   * it will be needed for a later step

![image](https://user-images.githubusercontent.com/9431420/90961157-36e6fb00-e4a7-11ea-87b7-5be7cb252cf5.png)

6. get top left & bottom right landscape borders (usually with custom actors)

![image](https://user-images.githubusercontent.com/9431420/90961227-b1b01600-e4a7-11ea-851a-876dcdd9a535.png)

7. get minimap corners (usually entities already exist)

![image](https://user-images.githubusercontent.com/9431420/90961285-13708000-e4a8-11ea-9dbe-46e1bc926db6.png)

![image](https://user-images.githubusercontent.com/9431420/90961289-1cf9e800-e4a8-11ea-9d67-31e1e3c6029d.png)

## Add map entry to squadmc_maps repository
Now that we have all necessary information extracted from the Squad SDK, let's add that information to the repository.

1. add entry in squadmc_maps -> mapdata.js

![image](https://user-images.githubusercontent.com/9431420/90961524-f0df6680-e4a9-11ea-8e94-ea27427147fd.png)

## Create lashkar.jpg (heightmap)
Now that the information has been added, we can run a npm script to tell us how to modify the raw heightmap in GIMP, so that it eventually overlaps properly with the minimap.

Additionally, we want to optimize the original heightmap as much as possible, as the original heightmap is way too large for mobile devices to handle.

1. execute "npm run info"
```
Lashkar Valley
    map dimensions: [4334,4334], scale: [1,1,1.5]
    orig heightmap: 4336x4336
scale heightmap to: 4336x4336
set canvas size to: 4334x4334
       with offset: 0x0
```
Now we just follow the steps as described, i.e. change the canvas to 4334x4334 with an offset of 0x0.

2. open heightmap in gimp

3. crop heightmap according to terminal output

![image](https://user-images.githubusercontent.com/9431420/90961583-369c2f00-e4aa-11ea-890c-d881d004ae2d.png)

12. adjust levels
Now the heightmap is cropped correctly, we want to optimize the level range.
Usually the levels of the exported heightmap are very broad, and only a small part of it is relevant to us. We therefore "crop" the initial levels to the range that we want.

**Important!** Don't forget to add this information to the mapdata object in the repository! This information is very important, so that SquadMC can properly convert the pixel values back to the actual height.

![image](https://user-images.githubusercontent.com/9431420/90961785-83ccd080-e4ab-11ea-92fe-eaacebd1d553.png)

![image](https://user-images.githubusercontent.com/9431420/90961638-a4e0f180-e4aa-11ea-82d3-fce982ab8708.png)

13. convert to rgb
Now that we have optimized the image levels, next is the color range. We could just export the grayscale jpg, but that is kind of a waste of information. You see, if we would export the image in grayscale, we only have one "color" to reflect the height. This means we only have 256 height "units", as each color can go from 0 to 255.
Additionally, each pixel would contain the same information, e.g. gray would be red = 127, green = 127, blue = 127. So we have three times the same information per pixel, which bloats the final file size.
Instead, we will change the image curves so that instead of going from black to white, we go from blue to black to red, while green is always 0. This way we have now split the original height scale from 256 units to 512 units!

![image](https://user-images.githubusercontent.com/9431420/90961665-dbb70780-e4aa-11ea-9610-f34641bd8cf4.png)

14. set curves (1)

![image](https://user-images.githubusercontent.com/9431420/90961794-9e9f4500-e4ab-11ea-9f0e-b3b3ed574726.png)

![image](https://user-images.githubusercontent.com/9431420/90961813-c68ea880-e4ab-11ea-99f6-5410206e49b4.png)

The image should now look like this:

![heightmap](https://user-images.githubusercontent.com/9431420/90962483-a6151d00-e4b0-11ea-9756-14006cc1445d.jpg)

15. save as jpg
   * set subsampling to 4:2:0 in Advanced Options to further reduce file size

![image](https://user-images.githubusercontent.com/9431420/90961730-33ee0980-e4ab-11ea-9c40-9640ef09cbf6.png)

16. undo curves

17. scale down to 4096

![image](https://user-images.githubusercontent.com/9431420/90961750-541dc880-e4ab-11ea-89aa-6c9f4a5bc4c6.png)

![image](https://user-images.githubusercontent.com/9431420/90961760-65ff6b80-e4ab-11ea-81e0-e8b6b7415a0f.png)

18. set curves (2)

![image](https://user-images.githubusercontent.com/9431420/90961821-d3ab9780-e4ab-11ea-842a-8118009b698e.png)