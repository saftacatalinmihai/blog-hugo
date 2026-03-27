---
title: "Raylib LCARS Frame"
date: 2026-03-26T18:19:44+02:00
images: 
  - "https://public.mihai-safta.dev/lcars_frame/lcars_frame.png"
seo:
  images: 
    - "https://public.mihai-safta.dev/lcars_frame/lcars_frame.png"
---

## [LCARS](https://en.wikipedia.org/wiki/LCARS) frame in [Raylib](https://www.raylib.com/)

Continuation from last [post](https://mihai-safta.dev/posts/lcars_elbow/) about designing the LCARS elbow. I continued with making the full frame and add some interactivity to it.

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
            document.getElementById('output').innerHTML += text + "<br>";
        },
        locateFile: function(path, prefix) {
            return 'https://public.mihai-safta.dev/lcars_frame/' + path;
        }
    };
  </script>
  
  <!-- Load the generated JS wrapper -->
  <script src="https://public.mihai-safta.dev/lcars_frame/lcars.js"></script>
{{< /rawhtml >}}

Full screen version available [here](https://public.mihai-safta.dev/lcars_frame/lcars.html)

Github Code [Repo](https://github.com/saftacatalinmihai/lcars)
