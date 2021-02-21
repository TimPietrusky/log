# Convert images

In order to not force the user to load big images, we convert all of them into small previews using [ImageMagick](https://imagemagick.org/):

```magick images\image.png -quality 1 -define webp:lossless=false -resize 660 images_minified\image.webp```

We could also use JPG, but with a quality of 1 a webp looks way better. 