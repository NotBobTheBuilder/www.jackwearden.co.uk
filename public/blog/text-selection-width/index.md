# Fixing oversized margins on text selection

Sometimes when selecting paragraph text on HTML documents, you may get a selection that takes up the full page width.
Typically, the paragraph you’ve selected has a `width` or `max-width` property, but the selection width is calculated at 100% of the width of the parent element or the page. It looks something like this:

(http://imgur.com/awh345A)

[You can see the demo here](/blog/text-selection-width/broken.html)

This, it turns out, is a pretty easy fix - you only need to add `overflow: hidden;` to the text container.

In my case, that’s the `article` block. The style now appears as:

    article {
      max-width: 400px;
      margin: 0 auto;
      overflow: hidden;
    }

and the solution:

(http://imgur.com/gtuio04)

[Working demo here](/blog/text-selection-width/fixed.html)
