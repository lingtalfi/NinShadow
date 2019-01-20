NinShadow
===============
2017-11-12



Ninshadow (short for ninja shadow) helps you providing visual insights to the
gui user when an ajax action is processing in the background.


Base principles
===================

When an ajax call is made, we want to enhance the user experience by prodiving an insight that
the app is doing something in the background.


There are two main types of shadow actions:

- localized action
- generic action



The localized action
----------------------

[![localized-loader.png](https://i.postimg.cc/ZKJJFn30/localized-loader.png)](https://postimg.cc/3WbM3KtH)

The localized action consist of displaying a small ajax loader, next to the gui control
the user just interacted with.

So for instance, if the user clicked a button, an ajax loader shows up next to the button
and disappears when the background action completes.



The generic action
----------------------

[![overlay-loader.png](https://i.postimg.cc/cHmn1H8j/overlay-loader.png)](https://postimg.cc/G842qbKJ)

The generic action is an overlay spanning the whole page,
and the loader is right in the center.

The user can't do nothing else but watch the loader disappear.




Implementation suggestion
==============================


Here is a simplified example sketch implementation.

```js



// somewhere in your js api
function ajaxRequest(){

    window.onRequestLoaderPrepare();
    window.onRequestLoaderStart();

    $.post(url, data, function (response) {
        window.onRequestLoaderEnd();
        // do other things
    });
}



// synopsis 1: generic action
$(".add-to-cart-btn").on("click", function(){
    ajaxRequest();
});

// synopsis 2: localized action
$(".add-to-cart-btn").on("click", function(){
    window.ninShadow = $(this).parent().find('.nin-shadow-loader');
    ajaxRequest();
});





```


The missing piece of this code is the following,
somewhere in your js code:

```js
(function () {
    var jNinShadow = null; // the current instance
    window.ninShadow = null; // allows external code to temporarily trigger the localized mode

    window.onRequestLoaderPrepare = function () {
        if (null !== window.ninShadow) { // do we want a localized action? (otherwise assume generic action)
            jNinShadow = window.ninShadow;
        }
    };
    window.onRequestLoaderStart = function () {
        if (null === jNinShadow) { // seems that we want a generic action
            jNinShadow = $("#nin-shadow");
        }
        jNinShadow.addClass('active');
    };
    window.onRequestLoaderEnd = function () {
        // resetting our variables
        jNinShadow.removeClass('active');
        jNinShadow = null;
        window.ninShadow = null;
    };
})();
```

And here is some example css code:


```css

// localized version
.nin-shadow-loader {
    display: none;
    border: 3px solid #d1d1d1;
    animation: spin 1s linear infinite;
    border-top: 3px solid #211313;
    border-radius: 50%;
    width: 24px;
    height: 24px;
    animation: spin 2s linear infinite;


    &.active {
        display: block;
    }
}


// generic version, don't forget to put
// a div#nin-shadow at the bottom of your page
#nin-shadow {
    display: none;
    position: fixed;
    opacity: 0;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 10000;
    background: rgba(236, 236, 236, 0.5);

    &.active {
        display: flex;
    }
}

```

