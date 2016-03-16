---
title: Creating Vertical Rhythm with CSS
date: 2016-03-14 15:04 UTC
tags: css, design
---

Vertical Rhythm is the pacing of our layout going up and down. Its a concept of typography that's been around since ever. When you see it, things feel right. If it's not there, something is somehow, kind of off...

Here's what a layout looks like when a vertical rhythm hasn't been established, or is not consistent.

<img src="../images/blog/vertical-no-rhythm.jpg" style="max-width:400px">

Notice how there's not much spacing between the text and it doesn't fall on any of the grid lines?

Here's what an established vertical rhythm looks like. There's some nice spacing between the lines of text and they are all placed neatly in the grid.

<img src="../images/blog/vertical-rhythm.jpg" style="max-width:400px">

The grid lines here are called the baseline grid. In print design, all the elements should rest on the lines. However there isn't really a concept for vertical rhythm built into CSS, so sometimes our elements will end up in the middle of the baseline grid... or somewhere close to it.

## Goals

Before we get into how to code a design with consistent vertical rhythm, this is why I'm so interested in this concept:

* **Looks Better.** Vertical rhythm helps with readability and comprehension. When things are visually organized on the page, the information is more accessible.
* **Code Faster.** If we're designing in the browser, we can fall back on a pattern, so that we write less code and spend less time tinkering.
* **Better Designs.** Whether we're designing in the browser or coding a mock up, we can spend more mental energy where it counts, knowing that we've appropriately handled this part of the design from the start.

## Setting up the CSS

I created [a codepen](http://codepen.io/austinnnnnnn/full/WwwmKP/) for our vertical rhythm example. Feel free to fork it and wipe out the CSS to see the examples for yourself - then add the CSS back in to see what we've accomplished.

First, for demonstration purposes we'll create a simple baseline grid:

```css
body {
  /* baseline grid is created by background */
  background: linear-gradient(#ccc, #eee 1px);
  background-size: 24px 24px;
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
}
```

The way to set the baseline for the grid is to match it to the line height of the body text. For our example, we'll use 16px as the baseline font size. Normally on the web, type shouldn't be smaller than that, or we risk making it unreadable. Allowing a line-height of 1.4 - 1.6 helps to give our text more breathing room. Doing the math: 16px * 1.5(line-height) = 24px. This is the leading. We'll use 24px (and some variations of it), for our spacing from here on out.

```css
body {
  /* ... */
  background-size: 24px 24px;
  /* ... */

  /* #2 setting up baseline font and line-height */
  font-size:16px;
  line-height:24px;
}
```

One way to keep images in line with the baseline is to make sure their height is a number divisible by 24. At the same time images are really fluid on the web. We'll set a max height to 480px, but on smaller screens it will be full width of the column... and also break the baseline grid. If you wanted to keep the image always matched to the baseline, you'd need javascript to continually resize it so its height is always evenly divisible by 24px... which I don't think anyone does.

```css
img {
  width:100%;
  height:auto;
  max-height:480px;
}
```

For typescale. I use [this modular scale tool](http://www.modularscale.com/) to visualize various scales and find the values it. For this example I'll go with the Perfect Fourth scale and match my fonts and their line-heights accordingly. I'm approximating the line-heights by first multiplying the font size in rem with 16, our base font size. Then I choose the smallest number that is greater in value and evenly divisible by 6. I'm choosing 6 because that's one quarter of the leading. Ideally I'd just use the leading, but it's too large a number. the h2 and h3 would both of the same line-height of 48px - which would be way too big especially for the h3.

```css
/*
3.157rem * 16 = 50.512
2.369rem * 16 = 37.904
1.777rem * 16 = 28.432
1.333rem * 16 = 21.328
*/

h1 { font-size: 3.157rem; line-height:54px; }
h2 { font-size: 2.369rem; line-height:42px; }
h3 { font-size: 1.777rem; line-height:30px; }
h4 { font-size: 1.333rem; line-height:24px; }

h1, h2, h3, h4 {
  margin-top:0px;
  margin-bottom:24px;
}
```

After the last step I changed the baseline grid to 12px to allow us more flexibility in laying out these elements. And everything should continue to fall near a baseline.

```css
body{
  /* ... */
  background-size: 12px 12px;
  /* ... */
}
```

We can use the leading, 24px, for padding and margins for any element we like, like lists, boxes, and button elements, while keeping the flow uninterrupted.

```css
li{
  margin-bottom:12px;
}
button{
  margin-bottom:24px;
  padding:12px 24px;
  /* extra style */
  background-color:plum;
  border:none;
}
.box {
  padding:12px 36px;
  background:aqua;
}
```

## How obsessed should we get?

I'm not sure. Notice if the headers go over one line in length, it becomes apparent that a sidebar column's paragraph text can go off baseline compared to the main column because the total height of the header element can fall into a 6px difference.

Should we use javascript to enforce all image heights stay within the leading? Should line-heights of header texts be recalculated if they run over one or two lines in length? Is there some perfect type scale with matching line-heights that will allow all elements of the page to sit perfectly on the baseline? I'm not sure.

To me, I think the most important highlights are using a modular type scale, and building spacing variables off the leading. A good practice for making spacing consistent would be to use only a few sizes:

```scss
line-height: $leading;

margin-bottom-xs:     $leading / 4
margin-bottom-small:  $leading / 2
margin-bottom-normal: $leading
margin-bottom-large:  $leading * 2
margin-bottom-xl:     $leading * 4
```

The same can be done for padding bottom or horizontal paddings and margins.

I like the idea of having 0px values for margin-top and padding-top whenever possible as a way of keeping the spacing flowing unidirectionally. This decreases the likelihood of a time when there is unexpected extra spacing between elements because one has a padding-top and another has a padding-bottom.  I came across this particular idea in this article, [Improving Layout With Vertical Rhythm](http://webdesign.tutsplus.com/articles/improving-layout-with-vertical-rhythm--webdesign-14070)

In researching this, I came across some great articles, and I'd also like to share some tools I've used in the past, or that I found this time around:

### Resources

* [Gutenberg - A meaningful web typography starter kit](http://matejlatin.github.io/Gutenberg/)
* [Modularscale](http://www.modularscale.com)
* [MegaType](http://megatype.studiothick.com/)
* [CSS Baseline: The Good, The Bad And The Ugly](https://www.smashingmagazine.com/2012/12/css-baseline-the-good-the-bad-and-the-ugly/)
* [Why is Vertical Rhythm an Important Typography Practice?](http://zellwk.com/blog/why-vertical-rhythms/)
* [Understanding Baseline Rhythm in Typography](http://www.sitepoint.com/understanding-baseline-rhythm-in-typography/)
* [Improving Layout With Vertical Rhythm](http://webdesign.tutsplus.com/articles/improving-layout-with-vertical-rhythm--webdesign-14070)
