---
title: "LCARS Raylib app - offline voice recognition & UI editing"
date: 2026-06-25T18:19:44+03:00
images: 
  - "https://public.mihai-safta.dev/lcars_voice_rec/lcars_voice.gif"
videos:
  - "https://public.mihai-safta.dev/lcars_voice_rec/lcars_voice.mp4"
seo:
  images:
    - "https://public.mihai-safta.dev/lcars_voice_rec/lcars_voice.gif"
  videos:
    - "https://public.mihai-safta.dev/lcars_voice_rec/lcars_voice.mp4"
---

## [LCARS](https://en.wikipedia.org/wiki/LCARS) voice recognition & UI editing

This is the continuation in the series of [LCARS](https://en.wikipedia.org/wiki/LCARS) app development:

1. [LCARS Elbow](https://mihai-safta.dev/posts/lcars_elbow/)
2. [LCARS Frame](https://mihai-safta.dev/posts/lcars_frame/)
3. [LCARS Text Editor](https://mihai-safta.dev/posts/lcars_texted/)

In the [last post](https://mihai-safta.dev/posts/lcars_texted/) I built a text editor from scratch for the Captain's-log use-case. This time I added three things to it: speech recognition, so you can dictate into the log instead of typing it out; proper cursor editing, so you can insert and delete anywhere in the text and not only at the end; and a drag-and-drop layout editor for moving the interface elements around.

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
            return 'https://public.mihai-safta.dev/lcars_voice_rec/' + path;
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
  <script src="https://public.mihai-safta.dev/lcars_voice_rec/lcars.js"></script>
{{< /rawhtml >}}

Works better in the full screen version available [here](https://public.mihai-safta.dev/lcars_voice_rec/lcars.html)
## "Computer..." — offline voice dictation

The headline feature: you can press the **VOICE INPUT** button (or hit `Ctrl+Space`), start talking, and watch your words land in the log. The button turns red and reads `RECORDING` while it listens.

The part I'm happiest about is that on the desktop build this runs **entirely offline**. No cloud API, no sending my microphone to someone else's server. It uses [Vosk](https://alphacephei.com/vosk/) with a small English model that the app downloads once on first run and then keeps locally. After that it works with the network cable unplugged.

The plumbing, roughly:

- [miniaudio](https://miniaud.io/) captures the mic at 16 kHz mono and pushes samples into a ring buffer from the audio callback.
- A background worker thread drains that buffer and feeds 100 ms chunks into the Vosk recognizer, so the UI thread never blocks on speech recognition.
- Vosk hands back two kinds of results: *partial* guesses while you're mid-sentence, and *final* text once you pause. The partials show up live in the status notification (`[Voice: "captain's log supplemental"]`), and each finalized chunk gets inserted into the editor at the cursor.

Because the recognized text goes in **at the cursor** (replacing the selection if you have one), dictation rides on exactly the same editing path as the keyboard — and the result is saved to the SQLite log just like typed text, so it survives a restart.

One honest caveat about the demo embedded above: shipping a ~40 MB speech model plus a native library into a browser tab isn't practical, so the **web build falls back to the browser's [Web Speech API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API)** instead of Vosk. That means the demo here is *not* the offline path — that one is cloud-backed and browser-dependent. The truly-local version is the native desktop app. Same C code, two recognizers behind one small interface, picked at compile time. English only, for now.

## Editing text anywhere: a gap buffer

In the previous post the text editor had an embarrassing limitation — you could only add or delete characters at the *end* of the text. I promised a [gap buffer](https://en.wikipedia.org/wiki/Gap_buffer) to fix that, and here it is.

So now you can actually edit like a real editor:

- Click anywhere to drop the cursor there, then insert or delete in the middle of the text.
- Move with the arrow keys, jump word-by-word with `Ctrl+←/→`, and hop to line start/end with `Home`/`End`.
- Select with click-and-drag or `Shift` + movement, and delete or overwrite the selection.
- Copy / paste with `Ctrl+C` / `Ctrl+V`.

The gap buffer keeps a movable "gap" in the character array right where the cursor is, so inserting and deleting are cheap no matter where you are in the text — you only pay to shuffle the gap when the cursor jumps. It's a classic trick, and it's a satisfying amount of bookkeeping to get exactly right by hand in C.

Logs can get longer than the screen now, so I added **scrolling**, with a draggable scrollbar, and the view snaps to follow the cursor as you type or dictate.

## Rearranging the bridge: a live layout editor

Hit `Ctrl+E` to toggle **edit mode**. Every element on screen sprouts two little handles: grab the top-left one to **move** it, grab the bottom-right one to **resize** it.

What makes this fall out almost for free is that the whole interface is just an array of generic elements — elbows, bars, buttons, the text editor, even the spinning 3D Earth are all the same `Element` struct under the hood, told apart by a `kind` field. So the layout editor doesn't care *what* it's dragging; it moves and resizes anything. Resize the Earth and its 3D render texture is rebuilt on the fly to match.

It turns the app into its own little LCARS designer — which is a nice place to end up, given the [whole series started](https://mihai-safta.dev/posts/lcars_elbow/) as a tool for designing a single LCARS elbow.

## Odds and ends

A couple of things that aren't user-facing but made building this much more fun:

- **Hot code reload.** The app logic lives in a shared library that's `dlopen`'d at startup. `Ctrl+Shift+R` recompiles it and swaps it in live, keeping all the application state — so I can tweak the editor or the layout and see it change without losing my place. (The web build skips this — it just runs the Emscripten main loop.)
- **One codebase, two targets.** The same C compiles to a native binary *and* to WebAssembly via Emscripten — which is why there's a playable demo embedded right here in the post.

As always, these features may sound basic — a text cursor, drag handles, dictation — but the point of the exercise is building them from scratch in [Raylib](https://www.raylib.com/), so each one is its own little rabbit hole. Next on the list: more UI element kinds - graphs, maps, etc... My goal is to make this usefull enough that I use it for myself every day: for journaling (Captain's Log), capturing data points, and viewing them. We'll see where it goes.

Github code [repo](https://github.com/saftacatalinmihai/lcars).
