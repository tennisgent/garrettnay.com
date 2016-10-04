+++
title = "@extend: One reason I love Sass"
date = "2013-12-16T19:19:50-07:00"
description = "Sass's @extend makes it ridiculously easy to share CSS rules between selectors."
+++

As I was working on my last post, I re-discovered a feature of [Sass](http://sass-lang.com) that
reminded me of why it's so great.

<!--more-->

I realized that when I was writing what the user should type (like <kbd>git pull</kbd>),
I should probably be using the `<kbd>` element instead of the regular code
backticks in Markdown, because [the element is specifically meant to indicate user input](http://devdocs.io/html/element/kbd).
According to the documentation, the `<kbd>` element is normally displayed in a
monotype font, but I was sad to find out that Octopress's default theme uses
some kind of CSS reset that makes it look just like normal text.

I tried hunting that reset down, but it was surprisingly hard to find. So I took
a different approach. I looked at the style rules for the `<code>` element, which
is what I wanted `<kbd>` to look like. This is what I found, in the generated
CSS:

```css
p code, li code {
    background: none repeat scroll 0 0 #FFFFFF;
    border: 1px solid #DDDDDD;
    border-radius: 0.4em;
    color: #555555;
    display: inline-block;
    font-size: 0.8em;
    line-height: 1.5em;
    margin: -1px 0;
    padding: 0 0.3em;
}
```

I wanted to get the `kbd` style to have the same rules. What to do? Well, as it
turns out, Sass has an extend feature, which [according to the documentation](http://devdocs.io/sass/index#extend),
tells the compiler "that one selector should inherit the styles of another
selector." That was exactly what I wanted—I wanted `kbd` to have all the same
styles as `code`.

Octopress uses Sass for its styling. It also makes things really easy by providing
an extra `_styles.scss` file inside the `sass/custom` directory, and the comments
inside explain that this file is compiled very last, so anything declared there
overrides previous rules. Or, more important in this case, they can *extend*
previous rules.

So all I had to do was add this simple rule to the file:

```scss
// sass/custom/_styles.scss
// This File is imported last, and will override other styles in the cascade
// Add styles here to make changes without digging in too much

// Style kbd tags the same as code tags
kbd {
	@extend code;
}
```

When I looked at the compiled code, I saw that Sass did exactly what it said
it would do:

```css
p code, p kbd, li code, li kbd {
    background: none repeat scroll 0 0 #FFFFFF;
    border: 1px solid #DDDDDD;
    border-radius: 0.4em;
    color: #555555;
    display: inline-block;
    font-size: 0.8em;
    line-height: 1.5em;
    margin: -1px 0;
    padding: 0 0.3em;
}
```

The `kbd` selectors were added right there to the same block as the `code`
selectors! How cool is that? It took just three lines of code (or one, if you
want to squish them together, but you shouldn't because that's what preprocessors
are for) to solve my problem, but after compilation, just a tiny bit was added
to the overall code. No bloat.

The bottom line is that Sass is awesome. I know that there are other CSS
preprocessors out there—[LESS](http://lesscss.org/) and [Stylus](http://learnboost.github.io/stylus/) probably being the biggest ones—and
they probably have analogous features, but Sass just happens to be the one I
went with first, so I can speak to how awesome it is. If your favorite
preprocessor can do the same thing and all sorts of other amazing things, more
power to you.

The real moral of the story here is that you should definitely be using a CSS
preprocessor if you aren't already. It makes coding CSS so much more pleasant
(dare I say fun?) and if used correctly can make your code a lot cleaner too.
