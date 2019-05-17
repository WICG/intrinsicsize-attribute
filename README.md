# `intrinsicsize` Attribute on Media Elements Explainer

There’s no way to do media that maintains aspect ratio, is proportional to the width of the screen, and doesn’t cause a user visible reflow. If we provide a way for the author to declare the intrinsic size of the image before the image has loaded, then we could allow them to specify just one dimension (e.g. the width) to a percentage or pixel value and compute the other dimension immediately without waiting for the image to load.

One edge case to keep in mind is when you want an image to size to its container, but the image is sometimes smaller than its container. In that case, the desired behavior is to have the image be its intrinsic size instead of stretching it.

## Proposed Solution
Add an `intrinsicsize` attribute: 

```HTML
<img intrinsicsize="400x300" style="width: 100%">`.
```

The `intrinsicsize` attribute tells the browser override the actual intrinsic size of the image with the width/height specified in the `intrinsicsize` attribute. Specifically, the image rasters at these dimensions and [naturalWidth/naturalHeight](https://html.spec.whatwg.org/multipage/embedded-content.html#dom-img-naturalwidth) on images would return the values specified in this attribute. 

On video elements, the video would raster at this size and the [videoWidth/videoHeight](https://html.spec.whatwg.org/multipage/media.html#dom-video-videowidth) on the video would return the value of the `intrinsicsize` attribute.

All other image size operations behave the same. So, for example:

* If no width/height are otherwise set, then the image dimensions are those specified by 'intrinsicsize'.
* If the `width` attribute is set on the image, then `intrinsicsize` would set the height to maintain the aspect ratio.
* If `width` and `height` attributes are both on the image, then `intrinsicsize` attribute’s value only affects the values of `naturalWidth`/`naturalHeight`, but not the rendered size of the image.

The sizes are in CSS Pixels.

This attribute applies on all image element types (including SVG images) and videos.

## FAQ
* **Why just image and video?** These are the only elements in https://html.spec.whatwg.org/multipage/indices.html#elements-3 that size to an external resource.

* **How does this relate to the `unsized-media` policy?** The idea behind the [unsized-media feature policy](https://github.com/WICG/feature-policy/issues/127) is to have a mode for the web where images don’t cause a reflow when the image data loads. This policy achieves that goal, but at the cost of making it extremely difficult to have images whose width is proportional to the size of their container and/or the screen while maintaining the aspect ratio. This proposal couples nicely with `unsized-media` in that it’s another way to set a size on an image/video. The unsized media policy just overrides the intrinsic size of the media element to be `300x150`. This lets authors override what that value should be.

* **How does this work with responsive images?** The `intrinsicsize` attribute sets the image's intrinsic aspect ratio. For an image element with a `srcset`, if:
    + a source is chosen using the `w` descriptor, then the result of evaluating the `sizes` attribute sets the image's intrinsic width, and its intrinsic height will be calculated by the intrinsic aspect ratio defined by `intrinsicsize` attribute;
    + otherwise, the `intrinsicsize` attribute sets the intrinsic width and the intrinsic height.

## Open questions
* **Does this let you do aspect-ratio for responsive layout videos or does that need an additional solution?** Aspect ratio on videos can be achieved in [terribly painful ways](https://alistapart.com/article/creating-intrinsic-ratios-for-video) today and [generally on any element](https://css-tricks.com/aspect-ratio-boxes/).

## Alternatives Considered

### aspectratio attribute
`<img aspectratio="4x3" style="width: 100%">`

* **Pro**: Simple to understand and implement
* **Con**:  Doesn't work for a common use case, where you want to constrain an image to be no bigger than its container, but not stretch it to be larger. That is, you want to set the image's width to min(width of container, intrinsic width of image), and you want to do so before downloading any image data.

#### Details
Using `intrinsicsize=""`, you can accomplish this like so:

```html
<div style="width: 75vw">
  <img src="my-image.jpg" intrinsicsize="400x300" style="max-width: 100%">
</div>
```

This gives the browser enough information to figure out the image's size, which will (correctly) end up as min(400px, 75vw), even before downloading any image data.

But if you tried to do this using `aspectratio=""`, there's no way for the 400 pixel measurement to enter the system. E.g. given:

```html
<div style="width: 75vw">
  <img src="my-image.jpg" aspectratio="4x3" style="width: 100%">
</div>
```

The width will be `75vw` always, which is incorrect. Or if the author does:

```html
<div style="width: 75vw">
  <img src="my-image.jpg" aspectratio="4x3" style="max-width: 100%">
</div>
```

Then the width will be correct eventually, but only after the image downloads; until then the browser doesn't know whether it should be `75vw`, or something less.

### width/height set the intrinsic size, New override attribute

```html
<img width="400" height="300" actualwidth="100%">
```

* Con: Doesn’t work with CSS or would need to also add a CSS property
* Con: Makes width/height mean something different than it does today. Not really backwards compatible.

### width/height set the intrinsic size, New override attribute

```html
<img intrinsicwidth="400" intrinsicheight="300" style="width: 100%">
```

* **Pro**: Allows you to only override one of width or height from the default of 300x150. Not clear this is really a pro. Is it useful to set only one dimension of the intrinsic size?
* **Con**: Requires setting two properties
