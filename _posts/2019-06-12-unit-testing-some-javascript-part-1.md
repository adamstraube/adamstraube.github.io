---
layout: single
comments: true
title: Unit Testing Some Legacy Javascript/JQuery - Part 1
date: 2019-06-12 13:05 +1000
---
As anyone who has done unit testing knows, implementing unit tests on legacy code is generally hard. I recently went through migrating a small JQuery script into a version control system, modularised it and implemented unit testing with the [Jest framework](https://jestjs.io) to verify it does what we expect it too. 

Implementing tests on your code cannot be overstated. Most of the benefits seem to repay in the medium to long term but some immediate ones:
 - Reduced likelyhood of bugs by testing many code paths
 - Reducing the risk of breaking older functions when implementing new features
 - Allows for easier bug fixing as tests help document the code

This first post prepares the ground work for implementing a test which we will tackle in the next post.

## Analysing the code

```javascript
<script>
$(document).ready(function() {
    $("#create-link").click(function() {
        var link = $("#link-input").val();
        var rewrite_link = "https://rewriteprefix.com/login?qurl="+link;
        
        $("#link-output").val(rewrite_link);
    });
});
        
```

The above code has a few obstacles for us in our quest. It is not versioned and is hard coded onto the page. Additionally the code is coupled via JQuery bindings to the page and the main logic is written into an anonymous function. 

## Refactoring:

### Break it down
We need to break this code down into discrete functions before going any further. This is so we can factor it out of the main page and introduce it as a dependency.

```javascript
<script>
$(document).ready(function() {

    function linkBuilder(link) {
      var rewrite_link = "https://rewriteprefix.com/login?qurl="+link;
      return rewrite_link;
    };


    $("#create-link").click( function() {
      var linkResult = linkBuilder($("#link-input").val());
      $("#link-output").val(linkResult);
    });
}); 
```

A number of things have happened to the above code:
 - All page specific bindings have been moved out into its own area
 - The logic of the function has been taken out of the anonymous function and moved to `function linkBuilder()`

### Separate logic into an external module and import

Now we can create a new JS file that will serve as our module. This can be then be added to Git or to the versioning system of your choice. I will refer to it as `linkBuilder.js` for the rest of this post.


```javascript
function linkBuilder(link) {
  var rewrite_link = "https://rewriteprefix.com/login?qurl="+link;
  return rewrite_link;
};
module.exports = linkBuilder;
```

The `module.exports` line declares that the `linkBuilder` function should be available when this file is imported.

This leaves us with the following code back on our front end page.

```javascript
<script>
import {
    linkBuilder
} from 'https://web/path/to/linkBuilder.js';

$(document).ready(function() {
    $("#create-link").click( function() {
      var linkResult = linkBuilder($("#link-input").val());
      $("#link-output").val(linkResult);
    });
}); 
```

The import statement added allows us to pull in the function we want from the module we have created.

## Next step

In the next post we will implement a unit test using the Jest testing framework. This will allow

