# Convert images

In order to not force the user to load big images, we convert all of them into small previews using [ImageMagick](https://imagemagick.org/):

```magick images\image.png -quality 1 -define webp:lossless=false -resize 660 images_minified\image.webp```

We could also use JPG, but with a quality of 1 a webp looks way better. 

# Convert videos

Use scripts/convert-video to convert the video into a webp-image:

```
./scripts/convert-video 20210214_RainbowGridWave_video_1
```

* The first frame from the first second of the video will be converted into JPG
* Then the JPG will be transformed into webp
* Then the JPG will be deleted as it's not needed anymore

For the script to work you need to install:

* ffmpeg
* magick