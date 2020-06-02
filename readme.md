# Final project for the 2020 course fundamentals of computer graphics: Adaptive rendering


The main objectives of this project are to integrate [this adaptive rendering algorithm][original] in the yocto-gl library, analyze his behavior with different scenes and try to improve it. This readme is organized in the following sections:

  - Integration with yocto
  - Comparison between adaptive sampling and the standard yocto sampling
  - My attempts to improve the algorithm
  - Final considerations



# Integration with yocto

The **Adaptive rendering algorithm** works on top of the render algorithm so it is pretty easy to add it on Yocto. In the original implementation, the author put his code in the `yocto/yocto_trace_adp.{h,cpp}` files and create a brand new application, the `apps/yscenetrace_adp.cpp` that has a similar command line interface as `apps/yscenetrace.cpp` except for the followings parameters:

| Parameter | Descpription |
| ------ | ------ |
| --quality, -q | (Mandatory) Set the target quality for the output image (in range 3:6) |
| --spp | Sample the image up to a specific sample per pixel. (Not settable with seconds)|
| --seconds | Sample the image for a specified time. (Not settable with spp)|

To integrate this work in the actual yocto release, instead of leave the adaptive sampling in his own application I shift it into the yscenetrace app, so with only one application is possible to use the adaptive sampling or the default. To understand if the user wants to use adaptive sampling, the application checks if the user sets the quality parameter, if not default sampling is used.

To make this change, I imported the `yocto/yocto_trace_adp.{h,cpp}` files, edit the `yocto/CMakeLists.txt` and finally edit the main function of `apps/yscenetrace_adp.cpp` in order to switch sampling strategy based on the quality parameters.

# Comparison between adaptive sampling and the standard yocto sampling

### Feature 1 test
For the first comparison test I taken the features 1 scene and i made , 
![512 sample - no adaptive sampling](out/readmeimg/NOadp_512_features.jpg)

As we can clearly see, There is some noise in the red sphere and a lots of noise in the green rabbit, the rest of the image is pretty clear. To make this render on my machine it took 15 minutes.




![512 sample - no adaptive sampling](out/readmeimg/adpFeaturesOld_512.jpg)
We have significant improvement in the critical areas of the scene, unfortunatly we have some noise spread in the rest of the image. To render this it took 27 minutes, almost the double of the time respect the no adaptive sample version.


### Kitchen test

Next i moved in a more complex scene, I tried with the kitchen:
![512 sample - no adaptive sampling](out/readmeimg/NOadp_1024_kitchen.jpg)

The image has lots of noise spread and not concentrate in some areas. To make this render on my machine it took 115 minutes.

![512 sample - no adaptive sampling](out/readmeimg/adpKitchenOld1024.jpg)

In this case the algorithm did not succed to enhance the quality of the image. The only improvement are on the glass of the microwave and the glass of the oven. For this image it took 106 minutes (little much speed than the no-adaptive).

# My attempts to improve the algorithm

### Merging the two technics

The features1 test inspired me to make a simple test, to reach the quality of a X sample image made without adaptive, i take a X/2 sample image made without adaptve and a X/4 sample image made with adaptive. Than i merge this two images by take the best pixels of every image. in order to choose the best pixels i used an image produced by the adaptive sampling application that shows which pixels received more sample, so by using that image I can select the best pixels in adaptive sampling image and put them on the other image. The script that merge images is `out/matlab test/merge.m`.

![512 sample - no adaptive sampling](out/readmeimg/NOadp_512_features.jpg)
This is the target image (15 minutes)



![512 sample - no adaptive sampling](out/readmeimg/04-merge.jpg)
And this is the final merge image that it took 14 minutes.

This experiment worked on this scene but it fail in a complex scene like the kitchen or the coffee machine. It seems that works only when the adaptive sampling works well. However the experiment highlights some weakness of the algorithm so I tried to analyze the code inside `yocto/yocto_trace_adp.{h,cpp}` and understand what can i made to improve that.


### Customizing the algorithm

Analyzing the code I understand why this adaptive sampling leaves noise spread in the image. Basically the algorithm starts with image at quality zero and than it tries to enhance the total image quality step by step (0.25). This means that zones that could reach high quality are penalized by zones hard to render. In order to rebalance the algorithm I made some changes:

  - The algorithm calculate a radius around a pixel based on the actual image quality, I add a bigger radius when the image is at a very low level of quality. 
  - The original algorithm send samples to a pixel until it reach the current quality step. I put a max limit to the sample to send that is the minimun number of samples taken by a pixel in the previous step.
  - There is a step in the algorithm where it sends samples to the neighbours of a pixel (that are inside the radius calculate previously) until it reach the same quantity of sample of the pixel. In my version the algorithm sends samples until it reach the half of the samples of the pixel.

Unfortunatly it seems that this version produces output images that are pratically similar to the original version of the algorithm and it is even slower, to produce the feaures 1 image at 512 samples it took 28 minutes

![512 sample - no adaptive sampling](out/readmeimg/adpFeatures512.jpg)

# Final considerations

The original adaptive algorithm can produce images with a better quality but it could be very slower compared to the non adaptive sampling, so it could be unpratical for a production use. The last attempt made me understand that probably what makes this implementation slow is the fact that the algorithm on every iteration creates lot of thread for pixels when instead the original algorithm just create threads at beginning of his execution. I suspect that an implementation that creates less thread or even recycle than could be much faster than the original.

[original]: <https://github.com/mkanada/yocto-gl>
