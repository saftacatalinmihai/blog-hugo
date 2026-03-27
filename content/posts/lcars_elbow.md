---
title: "Raylib LCARS Elbow designer"
date: 2026-03-18T18:19:44+02:00
---

## [LCARS](https://en.wikipedia.org/wiki/LCARS) Elbow designer in [Raylib](https://www.raylib.com/)

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
        consle.log(text);
    // document.getElementById('output').innerHTML += text + "<br>";
    }
};
</script>

<!-- Load the generated JS wrapper -->
<script src="/lcars.js"></script>
{{< /rawhtml >}}

Just a fun little project for designing LCARS elbows.
Click 'd' key for debug view.
Code: <https://github.com/saftacatalinmihai/lcars/blob/main/lcars.c>
