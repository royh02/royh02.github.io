---
layout: post
permalink: "/posts/cs180-proj1/"
# optional alternate title to replace page.title at the top of the page
alt_title: "CS 180 Project 1"

# optional sub-title below the page title
sub_title: "Roy Huang (FA23)"

# optional intro text below titles, Markdown allowed
introduction: |
  ## Introduction

  [Sergei Mikhailovich Prokudin-Gorskii](http://en.wikipedia.org/wiki/Prokudin-Gorskii) (1863-1944) [Сергей Михайлович Прокудин-Горский, to his Russian friends] was a man well ahead of his time. Convinced, as early as 1907, that color photography was the wave of the future, he won Tzar's special permission to travel across the vast Russian Empire and take color photographs of everything he saw including the only color portrait of Leo Tolstoy. And he really photographed everything: people, buildings, landscapes, railroads, bridges... thousands of color pictures! His idea was simple: record three exposures of every scene onto a glass plate using a red, a green, and a blue filter. Never mind that there was no way to print color photographs until much later -- he envisioned special projectors to be installed in "multimedia" classrooms all across Russia where the children would be able to learn about their vast country. Alas, his plans never materialized: he left Russia in 1918, right after the revolution, never to return again. Luckily, his RGB glass plate negatives, capturing the last years of the Russian Empire, survived and were purchased in 1948 by the Library of Congress. The LoC has recently digitized the negatives and made them available on-line.

# optional call to action links
actions:
  - label: "Project Link"
    icon: arrow-right # references name of svg icon, see full list below
    url: "https://inst.eecs.berkeley.edu/~cs180/fa23/hw/proj1/"

image: # URL to a hero image associated with the post (e.g., /assets/page-pic.jpg)

# post specific author data if different from what is set in _config.yml
author:
  name: Roy Huang

comments: true # disable comments on this post
---

## Color Channel Alignment

For the image plates, they are photographed in a way such that **3 color channels** (blue, green, red respectively) are separate as greyscale. See the below examples for reference. This allows the photographer to capture 3 channel data, and recombine them later on to form the colored picture.

![image](/assets/images/cs180/proj1/cathedral_split.jpg)![image](/assets/images/cs180/proj1/monastery_split.jpg)

The big challenge here is that due to this technique, the 3 channels are photographed separately, so they could be slight misalignments in photo positioning. Thus, the goal of this project is to find a programmatic method of aligning the channels.

**Here are the steps:**

### 1. Similarity Scoring

I ended up using the Sum of Squared Difference scoring method for similarity comparison. I tested 2 more scoring functions, Normalized Cross Correlation (NCC) performed at most as good as SSD, so it was a worse option.
I also explored Structural Similarity Index Measure (SSIM). It performed poorly and extremely slowly, so I didn't use this either.

### 2. Image Pyramid

For smaller JPG images, just exhaustive searching for the best similiarity scoring displacement was fine in terms of performance. However, for the larger TIFF images, I employed image pyramids to essential create multiple resolutions of the image, and perform exhaustive search only on the lowest resolution, and every level above, I keep the previously found best displacement, and search around the averaged pixels from the downscaling process.

**Chosen Parameters:**

- Base Case Search Range (`x` and `y`): `[-25, 25]`
- Subsequent searches: `[-5, 5]`
- Depth: 4
- Rescale Factor: 0.5 per level

The base case search range was just determined via trial-and-error. The choice of smaller subsequent searches was because afterwards, it is only necessary to search the pixels that were averaged in the downscaling process, and it helped a lot with speedup. Since it was a downscale of 0.5, a 2x2 pixel grid is averaged, so I chose 5x5 search area just to be safe. A depth of 4 gave generally decently sized images at the highest level of the pyramid.

For JPG images, I set the depth to be 1 to skip using the pyramid since they were already small and downscaling would destroy useful features of the images.

### 3. Cropping

For the images, since they had pesky black borders, I ordered for 1/16th of the height of the images to be cropped on all 4 sides of all the channel images. This helped with alignment since they interfered with the similarity score calculations as the border similarity doesn't do anything.

An exception is Emir. Somehow cropping less (1/19) just performed better. This could be due to the fact that Emir's alignment based on the background was decently useful as the clothing was complex enough that the background may help push the better alignment's SSD score to be better.

<div class="inline-block" style="margin: auto;">
    <img src="/assets/images/cs180/proj1/emir_bad.jpg" width="45%">
    <figcaption inline>1/16 Cropping</figcaption>
    <img src="/assets/images/cs180/proj1/emir.jpg" width="45%">
    <figcaption inline>1/19 Cropping</figcaption>
</div>

### 4. Color Base Alignment

For the color base, I chose green, which means that for the alignment, I aligned both red and blue onto the green channel as a reference point. This didn't really matter for most images, but for Emir, since the clothing was mostly blue and yellow, so there was a stark contrast between the pixel intensity values for the red and blue channels, which caused alignment to be iffy.

### 5. Edge Detection & Alignment

A big issue that arose out of a lot of images was that if an image was predominantly one particular color (a lot of grass or someone wearing blue clothes), it could be hard to use SSD as a similarity score since the pixel values actually differ by quite a bit between channels as a result. In comes edge detection, which by producing a greyscale image with object edges, alignment ended up being much better.

Here I used the Sobel filter for edge detection, and with the resulting images, I aligned the channels and obtained displacement vectors which were then applied onto the original images.

<img src="/assets/images/cs180/proj1/emir_edge.jpg" width="100%">

### 6. General Alignment

The general alignment using the displacement vectors from the previous parts is done using `np.roll` which produces some artifacts like the below. However, generally the overflow is not too serious, and as an extension, a final cropping can be done to trim this part off, or instead of rolling the channel over, pad with some predetermined value like 0 to prevent weird colors at the borders.

<img src="/assets/images/cs180/proj1/sculpture1.jpg" width="80%">

### Appendix

#### Images

<img src="/assets/images/cs180/proj1/emir.jpg" width="80%">
<img src="/assets/images/cs180/proj1/church1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/door1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/harvesters1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/icon1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/lady.jpg" width="80%">
<img src="/assets/images/cs180/proj1/melons.jpg" width="80%">
<img src="/assets/images/cs180/proj1/cathedral1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/monastery1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/onion_church1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/sculpture1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/self_portrait1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/three_generations1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/tobolsk1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/train1.jpg" width="80%">

#### Additional Images From the Library of Congress Collection

<img src="/assets/images/cs180/proj1/sunrise1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/shack1.jpg" width="80%">
<img src="/assets/images/cs180/proj1/door1.jpg" width="80%">

#### Displacement Vectors

Recall that I used green alignment, so the displacement will be for blue and red.

| Picture           | Blue Displacement | Red Displacement |
| ----------------- | ----------------- | ---------------- |
| monastery         | [3, -2]           | [6, 1]           |
| church            | [-29, -3]         | [37, -3]         |
| three_generations | [-51, -11]        | [61, -5]         |
| melons            | [-85, -13]        | [91, -5]         |
| onion_church      | [-43, -49]        | [51, 13]         |
| sunrise (Added)   | [-5, 0]           | [7, 0]           |
| train             | [-45, -5]         | [45, 29]         |
| tobolsk           | [-3, -3]          | [4, 1]           |
| icon              | [-45, -21]        | [43, 3]          |
| cathedral         | [-5, -2]          | [7, 1]           |
| door (Added)      | [-2, 1]           | [7, 1]           |
| shack (Added)     | [-6, -1]          | [7, 0]           |
| self_portrait     | [-75, -27]        | [101, 13]        |
| harvesters        | [-69, -21]        | [59, -5]         |
| sculpture         | [-27, 13]         | [109, -21]       |
| emir              | [-53, -29]        | [61, 21]         |
| lady              | [-61, -13]        | [59, 5]          |
| melons            | [-85, -13]        | [91, -5]         |
