---
title: "LCARS text editor"
date: 2026-04-21T18:27:09+03:00
images: 
  - "https://public.mihai-safta.dev/lcars_texted/lcars_texted.gif"
seo:
  images: 
    - "https://public.mihai-safta.dev/lcars_texted/lcars_texted.gif"
---

## [LCARS](https://en.wikipedia.org/wiki/LCARS) Text editor in [Raylib](https://www.raylib.com/)

This is the continuation in the series of [LCARS](https://en.wikipedia.org/wiki/LCARS) app development:

- [LCARS Elbow](https://mihai-safta.dev/posts/lcars_elbow/)
- [LCARS Frame](https://mihai-safta.dev/posts/lcars_frame/)

This time I focused on implementing a text editor from scratch in Raylib.

This is for something like the personal log / Captain's log use-case.

{{< rawhtml >}}
    <canvas id="canvas" style="
        max-width: 100%;
        max-height: 100%;
        display: block;"></canvas>
  <script>
    // Configuration for the Emscripten module
    var Module = {
        canvas: document.getElementById('canvas'),
        print: function(text) {
            // document.getElementById('output').innerHTML += text + "<br>";
        },
        locateFile: function(path, prefix) {
            return 'https://public.mihai-safta.dev/lcars_texted/' + path;
        }
    };

    // window.onkeydown = function(e) {
    //     if(e.keyCode == 32 && e.target == canvas) {
    //         e.preventDefault();
    //         return false;
    //     }
    // };
  </script>

  <!-- Load the generated JS wrapper -->
  <script src="https://public.mihai-safta.dev/lcars_texted/lcars.js"></script>
{{< /rawhtml >}}

Works better in the full screen version available [here](https://public.mihai-safta.dev/lcars_texted/lcars.html)

## Features

- Mouse over text area to allow editing text
- Insert text - only at the end for now... Will work on a [gab buffer](https://en.wikipedia.org/wiki/Gap%20buffer) or something in the future to allow moving the cursor and edit text anywhere.
- Text selection forwards and backwards with mouse click and drag
- Delete from the end with backspace key - holding down backspace will delete text with a delay
- Delete selected text with backspace or delete key
- Copy paste with Ctrl+C and Ctrl+V
- Using the native app (not for web yet), saving text into a local DB (sqlite)
  - restarting the app, retains the saved text
- Rotating Earth, before it was just a sphere wireframe.

These feature may seem basic (for the text editor for instance), but remember I'm implementing this from scratch in Raylib, so it's a start...

Works best on desktop. Did not look into making it mobile friendly yet.
