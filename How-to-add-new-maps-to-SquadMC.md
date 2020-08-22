Adding new maps to SquadMC is quite involved. As this is a web app with the goal of supporting mobile devices such as smartphones, we have to add a few optimization steps in order to get the performance where we want it to be. This includes splitting the image into smaller tiles and optimizing the heightmap.

The following steps will guide you to add the map "Lashkar Valley" to SquadMC.

## Contents

* [Export minimap & heightmap from Squad SDK](#export-minimap--heightmap-from-squad-sdk)
* create minimap layer
* create heightmap.jpg
* create heightmap layer

## Export minimap & heightmap from Squad SDK

### Open Squad SDK
   * it will be needed for a later step

### export minimap
![image](https://user-images.githubusercontent.com/9431420/90961022-21bd9c80-e4a6-11ea-82da-935d969a5f94.png)

### load AAS or RAAS layer of target map
![image](https://user-images.githubusercontent.com/9431420/90961057-73662700-e4a6-11ea-9375-ee5b422fe7f9.png)

### export heightmap
![image](https://user-images.githubusercontent.com/9431420/90961104-ccce5600-e4a6-11ea-9da4-efba4f9f8f47.png)

### write down landscape scale
   * it will be needed for a later step

![image](https://user-images.githubusercontent.com/9431420/90961157-36e6fb00-e4a7-11ea-87b7-5be7cb252cf5.png)

### get top left & bottom right landscape borders (usually with custom actors)
![image](https://user-images.githubusercontent.com/9431420/90961227-b1b01600-e4a7-11ea-851a-876dcdd9a535.png)

### get minimap corners (usually entities already exist)
![image](https://user-images.githubusercontent.com/9431420/90961285-13708000-e4a8-11ea-9dbe-46e1bc926db6.png)

![image](https://user-images.githubusercontent.com/9431420/90961289-1cf9e800-e4a8-11ea-9d67-31e1e3c6029d.png)

## Add map entry to squadmc_maps repository
Now that we have all necessary information extracted from the Squad SDK, let's add that information to the repository.

### add entry in squadmc_maps -> mapdata.js

![image](https://user-images.githubusercontent.com/9431420/90961524-f0df6680-e4a9-11ea-8e94-ea27427147fd.png)

## Create lashkar.jpg (heightmap)
Right now, the heightmap looks pretty grey:

![heightmap_raw](https://user-images.githubusercontent.com/9431420/90963083-5422c600-e4b5-11ea-8c62-6cdeca1cbcd7.jpg)

What we want to do now is crop the heightmap to the required dimensions, and optimize the contrast.

As we have added all the minimap & heightmap information to the code, we can run a npm script to tell us how to modify the raw heightmap in GIMP, so that it eventually overlaps properly with the minimap.

### execute "npm run info"
```
Lashkar Valley
    map dimensions: [4334,4334], scale: [1,1,1.5]
    orig heightmap: 4336x4336
scale heightmap to: 4336x4336
set canvas size to: 4334x4334
       with offset: 0x0
```
Now we just follow the steps as described, i.e. change the canvas to 4334x4334 with an offset of 0x0.

### open heightmap in gimp

### crop heightmap according to terminal output

![image](https://user-images.githubusercontent.com/9431420/90961583-369c2f00-e4aa-11ea-890c-d881d004ae2d.png)

### adjust levels
Now the heightmap is cropped correctly, so the next step is optimizing contrast.
Usually the levels of the exported heightmap are very broad, making the image pretty "grey", as only a small part of the greyscale range is relevant to us. We therefore "crop" the initial image levels to the range that we want.

![image](https://user-images.githubusercontent.com/9431420/90961785-83ccd080-e4ab-11ea-92fe-eaacebd1d553.png)

![image](https://user-images.githubusercontent.com/9431420/90961638-a4e0f180-e4aa-11ea-82d3-fce982ab8708.png)

**Important!** Don't forget to add this information to the mapdata object in the repository! This information is very important, so that SquadMC can properly convert the pixel values back to the actual height.

The image should now look like this:

![heightmap_cropped_leveled](https://user-images.githubusercontent.com/9431420/90963182-022e7000-e4b6-11ea-87fe-ee5a4d2bd4d5.jpg)

### convert to rgb
Now that we have optimized the image levels, next is the color range. We could just export the grayscale jpg, but that is kind of a waste of information. You see, if we would export the image in grayscale, we only have one "color" to reflect the height. This means we only have 256 height "units", as each color can go from 0 to 255.
Additionally, each pixel would contain the same information, e.g. gray would be red = 127, green = 127, blue = 127. So we have three times the same information per pixel, which bloats the final file size.
Instead, we will change the image curves so that instead of going from black to white, we go from blue to black to red, while green is always 0. This way we have now split the original height scale from 256 units to 512 units!

![image](https://user-images.githubusercontent.com/9431420/90961665-dbb70780-e4aa-11ea-9610-f34641bd8cf4.png)

### set curves (1)

![image](https://user-images.githubusercontent.com/9431420/90961794-9e9f4500-e4ab-11ea-9f0e-b3b3ed574726.png)

![image](https://user-images.githubusercontent.com/9431420/90961813-c68ea880-e4ab-11ea-99f6-5410206e49b4.png)

The image should now look like this:

![heightmap](https://user-images.githubusercontent.com/9431420/90962483-a6151d00-e4b0-11ea-9756-14006cc1445d.jpg)

### save as jpg
   * set subsampling to 4:2:0 in Advanced Options to further reduce file size

![image](https://user-images.githubusercontent.com/9431420/90961730-33ee0980-e4ab-11ea-9c40-9640ef09cbf6.png)

## generate minimap layer tiles
Now that we have the heightmap itself done, we can worry about the actual map layer. For this we have to take the exported minimap, and split it into tiles using a custom GIMP filter.

The filter can be found here: https://gist.github.com/Endebert/917e566846c482a8a3696ef406434ad1

### open minimap in GIMP

### execute the "leaflet" filter

![image](https://user-images.githubusercontent.com/9431420/90962568-79153a00-e4b1-11ea-9e65-901c31517441.png)

![image](https://user-images.githubusercontent.com/9431420/90962578-86322900-e4b1-11ea-8dd1-23ebe20c04a2.png)

## generate heightmap layer tiles
In order for users to also have a heightmap layer to look at (as we don't use the lashkar.jpg file as an overlay), we have to create another set of tiles just like we did with the minimap, but now we overlay the heightmap on top of the minimap first.

### open heightmap in GIMP

### crop the heightmap like we did before

### set the levels like we did before

### convert heightmap to rgb like we did before

### set curves (2)
Just like we changed the image curves in a previous step, we do almost the same here, but additionally, we set the green channel to cover the mid range.

![image](https://user-images.githubusercontent.com/9431420/90961821-d3ab9780-e4ab-11ea-842a-8118009b698e.png)

The heightmap should now look like this

![heightmap_cropped_leveled_cuved](https://user-images.githubusercontent.com/9431420/90962814-2d639000-e4b3-11ea-9c3b-08be5b012b84.jpg)

### scale the heightmap to 4096x4096
All minimaps have a size of 4096x4096, and we want to put the heightmap on top of it, so we scale the heightmap to match the minimap dimensions.
**NOTE:** Based on the map you're extracting, the heightmap might be smaller or larger than the minimap
**NOTE:** If the heightmap is not square, make scale it in a way so that the longest side becomes 4096px.

![image](https://user-images.githubusercontent.com/9431420/90961750-541dc880-e4ab-11ea-89aa-6c9f4a5bc4c6.png)

![image](https://user-images.githubusercontent.com/9431420/90961760-65ff6b80-e4ab-11ea-81e0-e8b6b7415a0f.png)

### open minimap in GIMP

### copy&paste heightmap into minimap and make it a new layer

### set the layer mode of the heightmap layer to _**Overlay**_
![image](https://user-images.githubusercontent.com/9431420/90962882-bb3f7b00-e4b3-11ea-83e8-298bf965bdac.png)

The image should now look like this:

![heightmap_cropped_leveled_cuved_overlay_interim](https://user-images.githubusercontent.com/9431420/90963040-fa220080-e4b4-11ea-8e6b-e023a92f87fc.jpg)

### convert the underlying minimap to grayscale
To improve contrast, we will now convert the minimap itself to grayscale. I usually use the "Color to Grey" function, as it gives better results than using "Desaturate".

So, make sure to have the minimap layer selected, and use the "Color to Grey" function:

![image](https://user-images.githubusercontent.com/9431420/90962932-15404080-e4b4-11ea-9d56-f46135069ca3.png)

Here are the "Color to Grey" setttings that I use:

![image](https://user-images.githubusercontent.com/9431420/90962965-62241700-e4b4-11ea-9430-051e85cc154c.png)

The image should now look like this:

![heightmap_cropped_leveled_cuved_overlay](https://user-images.githubusercontent.com/9431420/90963007-b62efb80-e4b4-11ea-8aee-d07da2ec8075.jpg)


### generate tiles
Just like we did with the minimap before, we are now creating tiles with the custom "leaflet" filter from our image

## And we're done! 🎉 
Now add the heightmap jpg, as well as the minimap and heightmap layer tiles to the squadmc_maps codebase (you specify the path in the mapdata object. make sure they match with the files locations!) and commit them under a new branch.

I'm looking forward to your pull requests!