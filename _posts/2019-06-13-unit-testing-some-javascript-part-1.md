---
layout: post
comments: true
title: Unit testing some legacy JavaScript/jQuery - Part 1
date: 2019-06-12 13:05 +1000
tags: Unit-testing Frontend JavaScript
---
As anyone who has done unit testing knows, implementing unit tests on legacy code is generally hard. Over the past 3 months I have been investigating how best to implement automated testing on a legacy codebase. With a view to get a good completed example for reference for the rest of the project, I chose a small javascript/jQuery application that would have a visible improvement with some simple unit tests. Along the way this application was modularised, moved into a version control system and implemented unit testing with the [Jest framework](https://jestjs.io) to verify it's expected functionality. 

The importance of implementing tests on your code cannot be overstated. There are many benefits to doing this, some of the most prominent are:
 - Reduces the likelyhood of bugs by testing many code paths and expected functionality
 - Reduces the risk of breaking older functions when implementing new features
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

The above code has a few obstacles for us in our quest. It is not versioned and is hard coded onto the page. This makes it impossible to track changes made, difficult to collaborate with others on and a high likelyhood of losing the application on accidental deletion. 

Additionally the code is coupled via JQuery bindings to the page and the main logic is written into an anonymous function. The code is married to the page and its functionality which means we cannot reuse the code elsewhere and anonymous function's cannot be called externally which is a requirement if we want to test the code.

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

Here is what I have done with the above code:
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

The import statement above allows the function to be pulled in from the module we have created. We now have a module with which to perform testing against.

## Next step

In the next post we will continue with implementing a unit test against this module using the Jest testing framework.

