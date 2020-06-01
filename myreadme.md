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

![512 sample - no adaptive sampling](out/readmeimg/NOadp_1024_kitchen.jpg)

In this case the algorithm did not succed to 

# My attempts to improve the algorithm

### Merging the two technics

The features1 test inspired me to make a simple test, to reach the quality of a X sample image made without adaptive, i take a X/2 sample image made without adaptve and a X/4 sample image made with adaptive. Than i merge this two images by take the best pixels of every image. in order to choose the best pixels i used an image produced by the adaptive sampling application that shows which pixels received more sample, so by using that image I can select the best pixels in adaptive sampling image and put them on the other image. The script that merge images is `out/matlab test/merge.m`.

![512 sample - no adaptive sampling](out/readmeimg/NOadp_512_features.jpg)
This is the target image (15 minutes)



![512 sample - no adaptive sampling](out/readmeimg/04-merge.jpg)
And this is the final merge image that it took 14 minutes.

This experiment worked on this scene but it fail in a complex scene like the kitchen or the coffee machine. It seems that works only when the adaptive sampling works well. However the experiment highlight some weakness of the algorithm so I tried to analyze the code inside `yocto/yocto_trace_adp.{h,cpp}` and understand what can i made to improve that.


### Customizing the algorithm



# Final considerations


[original]: <https://github.com/mkanada/yocto-gl>










### Installation

Dillinger requires [Node.js](https://nodejs.org/) v4+ to run.

Install the dependencies and devDependencies and start the server.

```sh
$ cd dillinger
$ npm install -d
$ node app
```

For production environments...

```sh
$ npm install --production
$ NODE_ENV=production node app
```

### Plugins

Dillinger is currently extended with the following plugins. Instructions on how to use them in your own application are linked below.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |


### Development

Want to contribute? Great!

Dillinger uses Gulp + Webpack for fast developing.
Make a change in your file and instantaneously see your updates!

Open your favorite Terminal and run these commands.

First Tab:
```sh
$ node app
```

Second Tab:
```sh
$ gulp watch
```

(optional) Third:
```sh
$ karma test
```
#### Building for source
For production release:
```sh
$ gulp build --prod
```
Generating pre-built zip archives for distribution:
```sh
$ gulp build dist --prod
```

```sh
cd dillinger
docker build -t joemccann/dillinger:${package.json.version} .
```
This will create the dillinger image and pull in the necessary dependencies. Be sure to swap out `${package.json.version}` with the actual version of Dillinger.

Once done, run the Docker image and map the port to whatever you wish on your host. In this example, we simply map port 8000 of the host to port 8080 of the Docker (or whatever port was exposed in the Dockerfile):

```sh
docker run -d -p 8000:8080 --restart="always" <youruser>/dillinger:${package.json.version}
```

Verify the deployment by navigating to your server address in your preferred browser.

```sh
127.0.0.1:8000
```

#### Kubernetes + Google Cloud

See [KUBERNETES.md](https://github.com/joemccann/dillinger/blob/master/KUBERNETES.md)


License
----

MIT


**Free Software, Hell Yeah!**

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)


   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>
