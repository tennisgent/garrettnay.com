+++
date = "2017-06-19T08:00:00-06:00"
title = "CSS Grid Is Great"
description = "CSS Grid allows us to do things that weren’t possible before."
+++

Here's a challenge for you. Let's say you're building a product page, and up at the top you have a container with three main pieces: one for the product name, one for an image of the product, and one for the product description. Your HTML looks something like the following:

<!--more-->

```html
<div class="container">
  <div class="item name">name</div>
  <div class="item image">image</div>
  <div class="item desc">desc</div>
</div>
```

On small screens, the three sections should appear in a simple column, with name at the top, followed by image, and then description. But on larger screens, the image should appear to the side and take the full height of the container, while the name and desription share the height of the container to the left of it. The mockup below illustrates what we're after:

{{< figure src="/images/post/css-grid/grid-example.png" title="The left version is what we want on small screens, the right on larger screens." alt="Mockup of a simple responsive grid design.">}}

The exact sizes of each section are not super important—basically we want to make them fluid—but there are three rules to the challenge:

1. You can use only features that were available in stable browsers in 2016.
2. You can't modify the DOM structure between the two screen sizes.
3. You can't duplicate any of the "content" (for example, to toggle visibilty between two different DOM structures with a class).

Can you do it?

Turns out it is possible, but it ain't pretty. If you're like the several people I showed this to, you might assume that flexbox is going to solve all your problems thanks to the miracle that is the [`order` CSS property](https://developer.mozilla.org/en-US/docs/Web/CSS/order). And while it's true that `order` can do some amazing things by separating the HTML structure from the order in which items appear in a flex container, in this case it's not enough.

See how on large screens, the image does come out of order, but it's separate in both dimensions. Flexbox by definition operates in a single dimension, so if you were trying to do this with just flexbox you'd need to have an outer flex container with the name and description on one side and the image on the other, and then an inner container that holds the name and description. But then how do you insert the image in between the name and description in the inner container on smaller screens?

Simple. You can't.

## Okay, Not Great: Flexbox and Floats

Not with flexbox alone, anyway. When I was working on a project that had a requirement similar to this challenge back in early 2016, I spent a day or more beating my head against a wall trying to get it to work with flexbox. I couldn't do it. I ended up going with a terrible duplicate-content solution at the time, using two copies of the image that alternately toggled on and off based on the screen size. But recently, as I was showing this around, someone reminded me of another CSS tool, one I hadn't used in so long I forgot it was even an option: the float.

After the era of table-based layouts, floating was the only real tool available in CSS for creating grid layouts. We laugh at it now because we know it was a hack and now we have better tools at our disposable. But floats, just like tables, still have legitmate uses, and you could argue that this particular situation is one of them because we're trying to float the image to the side.

So here you have my solution that combines flexbox with floating:

```css
.container {
  display: flex;
  flex-direction: column;
  height: 90vh;
}

.item {
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: sans-serif;
  font-size: 2em;
}

.name {
  order: 1;
  flex: 1;
  margin-bottom: 0.5rem;
  background-color: mediumaquamarine;
}

.image {
  order: 2;
  flex: 4;
  margin-bottom: 0.5rem;
  background-color: orangered;
}

.desc {
  order: 3;
  flex: 5;
  background-color: steelblue;
}


@media screen and (min-width: 40em) {
  .container {
    display: block;
  }

  .name, .desc {
    width: 40%;
  }

  .name {
    height: 20%;
  }

  .image {
    float: right;
    width: calc(60% - 0.5rem);
    height: 100%;
  }

  .desc {
    height: calc(80% - 0.5rem);
  }
}
```

<p data-height="388" data-theme-id="0" data-slug-hash="yXaWPd" data-default-tab="css,result" data-user="garrettnay" data-embed-version="2" data-pen-title="Responsive Grid: Using Floats and Flexbox" class="codepen">See the Pen <a href="https://codepen.io/garrettnay/pen/yXaWPd/">Responsive Grid: Using Floats and Flexbox</a> by Garrett Nay (<a href="https://codepen.io/garrettnay">@garrettnay</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

(Confession: for this version I had to swap the name and the image in the HTML to make it work. But the structure stays consistent between small and large screens, so it still follows the rules.)

I suppose it's not terrible—in fact, if I didn't know any better, I would say it's pretty good. But if you think about it, it's not all that great. One immediately noticeable problem is creating a consistent gutter while still having fluid percentage-based sizes for the sections. That 0.5rem gutter is all over the place even though it's essentially acting like a single value, which means that updating it would be error-prone. We'd have to use some kind of preprocessing to store that in a variable. (We could use a CSS variable, but remember, this is early 2016 we're talking about, so IE/Edge users would be out in the cold.)

Changing sizes of each piece would also be a pain. This is a simplistic example so it doesn't seem so bad, but imagine each element having at least five more style rules. Now imagine you need to come back to this stylesheet and tweak the width or height of one of these items. How hard would it be to find everything that needs to be updated to accommodate the change? Harder than it should be.

Or even worse, what if you had to add yet another piece to the grid, like a star rating? Perhaps it needs to appear below the image on large screens but between the name and the image on small screens. That may be a contrived example, but the point is I don't think it would take very long before you find that this float-flex solution breaks down.

Thankfully we have a better way now. If I haven't already made it obvious by the title of the article, I'm talking about CSS Grid. This is the real deal.

## Grid Saves the Day (Or At Least Makes It Much, Much Better)

I'm not here to give you a tutorial on how to use grid. For that, I recommend you check out [the guides on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout), [the examples and tutorials by Rachel Andrew](https://gridbyexample.com/), and [the experiments and examples by Jen Simmons](http://labs.jensimmons.com/). My main goal with this article is to show how CSS Grid improves how we do things and explain why I'm so excited about it.

Going back to the above challenge, here is my solution using CSS Grid:

```css
.container {
  display: grid;
  height: 90vh;
  grid-gap: 0.5em;
  grid-template-rows: 1fr 4fr 5fr;
  grid-template-areas:
    "name"
    "image"
    "desc";
}

@media screen and (min-width: 40em) {
  .container {
    grid-template-rows: 1fr 5fr;
    grid-template-columns: 2fr 3fr;
    grid-template-areas:
      "name image"
      "desc image"
  }
}

.item {
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: sans-serif;
  font-size: 2em;
}

.name {
  grid-area: name;
  background-color: mediumaquamarine;
}

.image {
  grid-area: image;
  background-color: orangered;
}

.desc {
  grid-area: desc;
  background-color: steelblue;
}
```

<p data-height="265" data-theme-id="0" data-slug-hash="WOxjLb" data-default-tab="css,result" data-user="garrettnay" data-embed-version="2" data-pen-title="Responsive Grid: Using CSS Grid" class="codepen">See the Pen <a href="https://codepen.io/garrettnay/pen/WOxjLb/">Responsive Grid: Using CSS Grid</a> by Garrett Nay (<a href="https://codepen.io/garrettnay">@garrettnay</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Check it out! Notice, first of all, that the aforementioned grid gutter is defined in a single place in the container with the [`grid-gap` property](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-gap). That combined with using the [`fr` unit](https://developer.mozilla.org/en-US/docs/Web/CSS/flex_value) to define the rows and columns means that if you ever need to change the size of the gutter, you only need to go to that one place, and everything else will accommodate the new value without overflowing the container.

What really blows me away about the grid version, however, is the fact that the whole definition for the grid container and all its members, including their relative sizes, is right there in the rules block for the container. Since we're using [named grid areas with `grid-template-areas`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-areas), all the individual grid items have to do is declare which grid area they belong to, and everything else Just Works™. So when it comes to different screen sizes, the only thing that needs different media query definitions is the container. And maintenance is much easier. Need to change the size of one of the items? Adjust the columns or rows in the container. Need to add an item? Add another area to the grid. It's all there in one place.

This is a pretty simple example, but I think it nicely illustrates why CSS Grid is something to be excited about. And it's only the beginning (seriously, check out those experiments by Jen Simmons!).

## Why I'm Excited About CSS Grid

In conclusion, here are some reasons why I think CSS Grid is so great:

- We can use CSS layout properties they way there were intended to an unprecedented level. Floats can be used to, well, float things. Flexbox can be used to arrange items in a single dimension. And grid can arrange entire pages and handle other two-dimensional layout concerns. No more using properties in hacky ways they weren't meant for.

- As I mentioned above, the fact that you can define the entire grid layout in the container seems revolutionary to me. CSS has never really had anything like that before. Of course, there are lots of ways to use CSS Grid, and individual grid items can have more granular control over their own placement if you need them to, but this new grid paradigm can truly change the way you think about layout in CSS.

- For the first time, it's probably actually easier to create a grid layout *without* a framework or preprocessor. I don't have a lot of experience with it yet so I can't say for certain, but [that's what the experts are saying](http://jensimmons.com/post/feb-28-2017/benefits-learning-how-code-layouts-css) and I believe it. CSS Grid has a bit of learning curve, sure ([check out this cute game to learn the basics](http://cssgridgarden.com/)), but so does your Bootstrap or Foundation or Lost Grid or any other of the millions of frameworks that exist. Don't get me wrong—those things are all great, but now that we have a grid solution baked right into the browser, there's some real power in that. And you no longer have to fight against any choices the framework has made. What if you want an odd number of columns instead of 12? Using Bootstrap? Forget about it. Using CSS Grid? Go nuts! Which brings me to my final point.

- CSS Grid enables us to do things in web design that weren't possible before. (I originally thought my example for this article was such a thing. I was wrong, but still.) Rachel Andrew and Jen Simmons, the great CSS Grid evangelists, emphasize this point a lot: We need to stop constraining ourselves design-wise by what we've already seen. Yes, CSS Grid makes implementing the ["holy grail" layout](https://en.wikipedia.org/wiki/Holy_Grail_(web_design)) easier than ever, but why stop there? Let's experiment and see what new things we can come up with. I think, in large part, we need to unlearn what we have learned about web design over the years. For us developers who work with designers that don't know CSS, I think it's partly our responsibility to show them what's possible. All those designs we used to tell them were impossible are no longer so. We need no longer be limited by old design patterns. I'm excited to see what we come up with.

I don't think I've ever been so excited by a CSS feature since I first learned CSS. And I don't believe that excitement is misplaced. CSS Grid is a big deal. Let's take advantage of it.
