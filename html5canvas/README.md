HTML5 Canvas
==================================

Solution to Uncaught SecurityError
----------------------------------

In many of the exercise, I kept seeing `Uncaught SecurityError` when trying to invoke `getImageData()`. A fix is to put your code in `index.html` and then run a server locally. Since I'm on a mac which has python automatically installed, you can run

    python -m SimpleHTTPServer

By default, the server is running on `0.0.0.0` port `8000`. 

The Game Loop & Processing User Input
-------------------------------------

See `requestAnimationFrame` from [Paul Irish](http://www.paulirish.com/2011/requestanimationframe-for-smart-animating/).

The game loop is a sequence of events that run continuously while an app or game is being used. `requestAnimationFrame` handles most of the heavy lifting in that it ensures that your app runs as close to 60 frames per second as possible while the app is being actively viewed.

Assuming we have already creating the functions we plan to call, a fleshed out game loop could look something like this.

    function draw() {
        // request to execute this function at the next earliest convenience
        requestAnimationFrame(draw);
        processInput();
        moveObjectsAndEnemies();
        drawAllTheThings();
    }

[Kibo.js](https://github.com/marquete/kibo) is a JS library for processing keyboard input. Kibo allows you to reference keys by their common names('a', '3', 'up') instead of their keycodes greatly simplifying your code. You can also attach events to pressing or releasing a key as well as modifier keys or wildcards.

    var k = new Kibo();
    k.down(['up', 'w'], function() {
        // Do something cool on the canvas
    });

    k.up(['enter', 'q'], function() {
        // Do other stuff.
    });


Inspiration
-----------

- [http://mrdoob.com/](http://mrdoob.com/)

