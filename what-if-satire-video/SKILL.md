---
name: "what-if-satire-video"
description: "Turn a narrated storyline into a 9:16 low-poly 3D satire video (\"What if X disappeared?\") with Day/Week counter, serif captions, music and SFX."
---

# What-If Satire Video

The user narrates a storyline ("what if Yo-Chi disappeared… day 1 the queue stays… week 1 the xmms and xdds are estranged…"). You turn it into a finished vertical MP4 in a fixed house style. Build it with three.js scenes rendered frame by frame in headless Chromium, add a generated soundtrack, and deliver the file with SendUserFile.

**Quality is the default, not an upgrade.** The user wants every video to look like a finished, premium short: detailed hero characters, cinematic lighting, bloom and colour grading, supersampled frames, readable overlays, and every shot checked at several moments before the full render. The Quality bar section says how.

## The house style (keep this consistent every time)
- 1080x1920, 30 fps, about 40–50 s final length. Render at SPEED=1.25 by default: write the timeline in comfortable seconds and let the renderer compress it.
- Crowds are low-poly blocky people: box torso, sphere head with hair cap, box limbs with hip/knee pivots, white shoes (rounded boxes look better than sharp ones). Crowds are dense and goofy. The characters the story is about get a detailed hero model instead (see Hero models), so whatever the user wants emphasised, such as a uniform, a hairstyle or a car, reads clearly in close-ups. Gags come from body language: freezing, '?' pop-ups over heads, slumping, sitting, sprinting, cheering.
- Blue-hour dusk sky by default. Night only for a 'then one night…' beat.
- Text overlays are HTML over the canvas:
  - Opening title in Crimson Pro serif, centered, with key names in yellow (#ffd35a). Example: "What if **Yo-Chi** and **Molly Tea** just… disappeared?"
  - HUD at top-left from shot 2 onward:
    - small caps label, e.g. TIME WITHOUT YO-CHI
    - big serif value: Day 1 / Week 1 / Month 1 / Day 47
    - a funny stat line, e.g. HOPE: 3%, XMMS UNACCOUNTED FOR: 2,400
    - a location pill, e.g. ▶ SOMERSET, SINGAPORE
  - Italic serif captions in the lower middle, max 2 short lines, each held about 3 s in timeline time.
  - End with a fade to black and an italic "Part 2?".
- Story shape: title → Day 1 setup → the vanishing (poof cloud plus a pale ghost rectangle on the wall) → people refusing to accept it → escalating time jumps (Day 2/3, Week 1/2, Month 1) with one absurd beat each → a twist or return at night → "Part 2?". Other premises (a war, a face-off, something that keeps growing, a trend taking over) follow the same escalation. Show the premise on screen: in a war, the fights are the centrepiece (2–3 action shots with brawl clouds, POW!/BAM! bursts and knocked-out stars, slapstick only), not one implied clash.
- Use the user's own slang and spelling exactly. If they say "xmm/xdd" or name a brand, use their words in captions from the first shot that's relevant. Spell real shop names correctly in plain lettering on signs. Never reproduce actual logos.
- Reference photos or videos the user sends are only for set layout, colours and materials. Never put real people from them into the video.
- No real, named people in the video at all: celebrities, influencers, executives or private individuals. That holds even as blocky figures, even when the user says they consented, and even for "a friend with the same name", because viewers will read the character as the real person. Offer a fictional stand-in with a role or a number ("the Grooming Inspector", "the two GOATs, #65 and #88") and keep the rest of the idea.
- Signature designs (uniform prints, liveries, kits) are redrawn as originals in the same spirit and colours, never copied, and never with the logo.

## Quality bar (the default for every video)
- **Look test first.** As soon as the hero character and one set exist, render 1–2 frames at production settings with the real HUD and a caption. Send them side by side as `look_test.jpg` and keep building the other sets while the user looks. Before the full render, check a one-still-per-shot sheet too.
- **Hero models.** Whatever the user wants emphasised (a uniform, a hairstyle, a car, a building) gets a detailed model and at least one close-up. Crowds stay blocky. The recipe is in Hero models below.
- **References and assets before modelling.** Research the real place, uniform or object: WebSearch, WebFetch on Wikipedia for layout and dimensions, image_search, or Bing Images in the built-in browser when the user's computer is connected. Then pull CC0 assets (see Setup). Get facts right in the HUD stats and captions. A real in-joke found in research often becomes the strongest beat (an ex-stewardess's "the higher your hair hump, the more senior you are" carried a whole episode).
- **Lighting.** Per set: one shadow-casting key with a 2048 map, a hemisphere fill, one to three point lights, and a rim or back light on the hero. Build each set in its own group at its own origin and toggle `G.visible` per shot, so a hidden set's lights drop out of the shaders. Give things that glow (screens, signs, bulbs, skylights) emissive values above 1 so they bloom. Keep point lights away from ceilings and glossy surfaces, where they blow out into white blobs.
- **Post-processing** (post.js): HalfFloat composer → BokehPass on close-ups only → UnrealBloomPass at half resolution → grade and vignette → OutputPass. Shots can override `bloom`, `exposure` and `dof`. Night beats get lower exposure and less bloom so they read as night.
- **Supersampling.** Render at `DPR=1.25` (1.5 when there's time) and let ffmpeg downscale with lanczos, encoding with `-preset slow -crf 16`. On SwiftShader, 4× MSAA costs more than DPR 1.25 and looks no better, so keep the composer's `samples` at 0.
- **Readable overlays.** Soft dark radial backdrops sit behind the HUD and the captions (`#hudbg` and `#capbg` in the template), so white text holds up over light walls and floors.
- **Effects that read as quality:** bloom on lights, screens and signs; water with real reflections (Reflector or Water, with a few blurred taps); instanced crowds in the hundreds or thousands; confetti, fireworks, sparkles and rain; procedural lit-window facades for skylines; slow push-ins, orbits and low-angle tracking shots; depth of field on the key close-up; colour or state changes shown as a sweep rather than a cut.
- **Cost control** (2 cores, CPU WebGL): a scene-wide `scene.environment` roughly doubles frame time, so put `envMap` only on small glossy materials (hair, metal, pearls, polished floors, car paint). Transparent glass with an env map is also expensive; MeshBasicMaterial at about 0.07 opacity looks nearly the same. Instancing cuts draw calls, but per-pixel shading dominates. Benchmark 40 frames of the heaviest shot before committing to a DPR.

## Hero models (what worked)
Build heroes in a `people.js` module from smooth shapes, painted textures and a few physical materials:
- **Body and clothes:** `loft()` (lib.js) sweeps elliptical rings `[y, rx, rz, cz]` that `smoothRings()` interpolates, so a fitted top, a waist, a sarong or a skirt is just a list of rings (for example hem .785/.18/.142, waist 1.03/.14/.108, bust 1.17/.158/.126, shoulders 1.335/.16/.10, neck 1.425/.058/.055). Arms are pivot groups: shoulders use rotation order XYZ, elbows YXZ so a bent forearm can swing inward for clasped hands. A small sphere at the shoulder pivot (radius about .04) hides the joint without puffing the sleeve. Long skirts deform each frame from the foot offsets, so walking reads without visible legs.
- **Prints:** paint a tileable 2048² canvas (draw every motif and crackle line nine times with `wrap9`) with a half-drop motif grid, wax-crackle lines and dot fills. Around a closed loft `repeat.x` must be a whole number, and loft v is in metres, so `repeat.y = 1 / circumference` keeps motifs square. Trims are TubeGeometry along `ellipseCurve()` (necklines, hems, cuffs) plus a thin ribbon down the front, with a border texture. Fabric is MeshPhysicalMaterial with roughness .6, sheen 1 and sheenRoughness .32.
- **Faces:** paint eyes (white almond, iris, highlight, liner flick), brows, lips and blush on a 2048×1024 canvas used as the head sphere's map. The front (+z) of a SphereGeometry is u=.25, so on that canvas the eyes sit near x 402 and 622, y 493, and the mouth near x 512, y 676. Add small ears and earrings; glasses are two tori and a bridge.
- **Hair:** a cap shell made from a sphere rotated so its pole sits at the bun, where the strands converge. Push the radius inside the head where skin shows, with a gradual transition (about .1) so the hairline is smooth instead of jagged. Use a strand canvas as both `map` and `bumpMap`, on MeshPhysicalMaterial with anisotropy .8 (rotation π/2), roughness about .48, no clearcoat, and little sheen or env. Too much specular turns black hair grey.
- **Volume that grows** (a beehive, an afro, a hat): a dome that follows the cap at its base, lifts away and ends in a hemispherical top. Widen it over the lower 35–55% of its height; a fast widening looks like a balloon on a stick. Rebuild the geometry in place when the size changes (a few thousand vertices is cheap), and add a squash-and-stretch overshoot (`springy()`) on growth beats.
- **State changes:** swap colours with `addWipe()` (lib.js), a downward sweep with a gold glow band, plus a few additive sparkles.
- **Proportions in this style:** head radius about .1 m with a 1.1× vertical stretch, 1–2 cm of neck showing, about 1.66 m tall before hair. Blocky crowd people are about 1.75–1.85 m.

## Workflow
0. **Ideas first when asked.** If the user asks for ideas, research first (facts, dates, in-jokes, local slang), then propose 2–3 concepts: full beats with HUD stats for the one you recommend, a two-line pitch for each other one, and sources. Wait for their pick. If they reply without picking (for example "focus on quality"), build the recommended one and say so in your first line. If a premise centres on a real, named person, pitch it with a fictional stand-in and give the reason in one sentence.
1. **Parse the narration into 7–10 shots.** Each shot gets: a timeline range (5.5–7.5 s), a HUD value and stat, 1–2 captions, a camera move, and the cast and actions. Write the shot list and send it to the user with SendUserMessage in one short message, then start building. Don't wait for approval unless the narration is genuinely unclear.
2. **Set.** If the location is new, ask once for 3–6 phone photos or short clips (both ends of the corridor or street, the key shopfronts, a crowded moment). You can also use WebSearch and image_search for reference. Contact-sheet any videos with ffmpeg (`fps=12/dur,scale=300:-1,tile=4x3`) and Read them. Rebuild the set from the reference template below: change the floor, the canopy/slat colours, the shopfronts (`shop(z,len,name,sub,signBg,signFg,insideColor)`), the right-side features, and the far-end landmark. If the location is the Somerset/Orchard walkway, reuse the template set as-is. For anything new, gather references and assets before modelling (see Quality bar). If you can't tell how the hero should look, ask 1–4 quick AskUserQuestion questions (outfit, hair, colours) first.
3. **Cast and shots.** Edit the `casts` and `SH` arrays. Reusable pieces:
   - `person(style)`, with styles adult / xmm / xdd; add new styles to STY as needed
   - `pose()`, which takes walk, speed, sit, armUp, headYaw, headPitch, slump, lie and cheer
   - `qpop()` for the '?' pop-ups
   - `makePuff()` + `setShop()` for vanishing and reappearing
   - props in the right hand: `cup`, `bowl`
   - `stall` (a cardboard knock-off shop) and `tumble` (tumbleweed)
   - `glowL` for the lights-come-back-on beat
   Every shot's `upd(t)` must set visibility and position for everything it shows, because `renderAt` hides all people each frame.
   Detailed heroes, per-set moods and per-shot `bloom`/`exposure`/`dof` overrides sit alongside these (see Hero models and post.js).
4. **Check stills before rendering.** Run `PAGE=film.html node render.js stills t1,t2,…` at three moments per shot (start, busiest moment, end). Walkers on looping lanes and people walking in can cross the camera or block the subject partway through a shot, and one still per shot won't show it. Use `DPR=1` for these checks because it's faster, and the production DPR only for look tests. hstack them, Read the result and fix problems. Common problems:
   - groups out of frame: portrait framing is narrow. At a 50° vertical FOV the frame is only about 29° wide, so anything more than about 14° off the look line is out; keep cast within about ±2.5 units of the look line at 10 units out
   - people sinking into seats: for a sitter, set `g.position.y≈.09–.11` and keep the hip x so the knees clear the bench edge
   - signs or props hidden behind the HUD (top-left, y 590–800) or the captions (y 1230–1400)
   - captions too long
   - foreground people blocking the subject: move the camera to one side, raise it above head height, or route walkers away from the line of sight
   - a climax subject that's small and dark (this has happened twice): bring it closer and light it with a rim light, torches or a pool of light
   - height gags against a line on a wall: put the camera at the line's height, otherwise parallax makes everyone in front of it look taller
   - title shots: keep subjects below the title band (y 440–760)
   - hotspots: point lights near ceilings or glossy surfaces bloom into white blobs, so lower them or cut their intensity
   - black hair or navy suits reading grey: too much clearcoat, sheen or env reflection
5. **Render.** One MP4 per shot and two parallel jobs started with nohup (see runall.sh). Background processes are stopped a few minutes after your reply ends, so stay in the turn and check with foreground `sleep 540` calls until every shot is done, then mux and deliver in the same turn. With the quality pipeline at DPR 1.25 on 2 cores, expect about 4.5–6 s per frame per job for simple interiors, 6–7.5 s for big sets with crowds, and 8–9 s for depth-of-field close-ups or packed cabins. A 43 s video (1,284 frames) takes about 70–80 minutes. Tell the user the ETA as a clock time, and send a one-line update if it slips by more than about 15 minutes. Balance the jobs by cost rather than by shot count, and finish with the heaviest shot split into two halves rendered in parallel (`shot7a` and `shot7b`; the split offset from the shot start must be a whole number of frames, i.e. a multiple of 1/24 s timeline at 1.25×). Each shot render loads the page fresh, so editing film.html or a module mid-render changes every shot that hasn't started yet; only make edits that are safe for all of them. Contact-sheet each shot as it finishes so a bad one can be re-rendered while the rest are still running. If they ask "check", report which shots are done.
6. **Music and SFX.** Edit the EVENTS in `music.py` to the shot times: poof at each vanish, pips for cups or '?' pops, whoosh for the tumbleweed, the night hush, and the shimmer + cheer at the return. Set SP to the same SPEED and DUR to the timeline end. Put every SFX on the exact beat it belongs to: boing for growth, thud for impacts, chime for an airport or PA moment, clicks for ratchets, tape measures and camera shutters. When flashes or pops are randomised in the page, replicate the page's RNG in Python to time them. Use `MUTE` to drop the bed for a standoff or a hush. Check the result with `showwavespic` and `volumedetect` (mean around −15 dB); the template normalises by RMS with a soft limiter, because normalising by peak lets one loud SFX make the whole track quiet.
7. **Mux.** Concat with the demuxer and `-c copy`, add the audio with `-map 0:v -map 1:a -c:v copy -c:a aac -b:a 192k -shortest -movflags +faststart`. Make a 16-frame contact sheet, Read it, and confirm every shot and caption is present. Then send it with SendUserFile (`display: render`). Describe what's in it in one or two lines, say honestly which shot is weakest, and offer one concrete tweak with its re-render time.
8. **Revisions.** Re-render only the shots that changed. Caption or sign changes affect every shot that shows them. A speed change means re-rendering everything and regenerating the music with the new SP. Split a single re-rendered shot into two halves so both cores work on it.

## Setup (once per session)
```bash
mkdir -p $SCRATCH/build && cd $SCRATCH/build && npm init -y >/dev/null
npm i three@0.160.0 @fontsource/crimson-pro@5.0.8 @fontsource/inter@5.0.16 >/dev/null
# playwright + chromium are preinstalled: do NOT run playwright install
```
Write film.html, render.js, runall.sh, post.js, lib.js and music.py from the templates below into that folder.

CC0 quality assets. The npm registry and raw.githubusercontent.com are reachable from the shell; most image hosts aren't.
```bash
mkdir -p assets && npm pack @pmndrs/assets >/dev/null 2>&1 && tar xzf pmndrs-assets-*.tgz
node -e "const fs=require('fs');for(const n of ['night','city','sunset','dawn','park','lobby']){const s=fs.readFileSync('package/hdri/'+n+'.exr.js','utf8');fs.writeFileSync('assets/'+n+'.exr',Buffer.from(s.slice(s.indexOf('base64,')+7,s.lastIndexOf(String.fromCharCode(39))),'base64'))}"
curl -s -o assets/waternormals.jpg https://raw.githubusercontent.com/mrdoob/three.js/r160/examples/textures/waternormals.jpg
```
The package's `textures/` and `normals/` folders use the same base64-in-JS format. Load an HDRI with `exrEnv()` from post.js and use it as an `envMap` on chosen materials (see Cost control).

## render.js
```js
// node render.js stills t1,t2,...  |  node render.js video out.mp4 tStart tEnd   (env: PAGE, PORT, SPEED, DPR, Q=query string)
// DPR supersamples: frames are captured at DPR size and downscaled with lanczos; stills come out at DPR size.
const { chromium } = require('playwright');
const http = require('http'), fs = require('fs'), path = require('path');
const { spawn } = require('child_process');
const root = __dirname;
const srv = http.createServer((q, r) => {
  const p = path.join(root, decodeURIComponent(q.url.split('?')[0]));
  if (!fs.existsSync(p)) { r.writeHead(404); return r.end(); }
  const ext = path.extname(p);
  r.writeHead(200, { 'content-type': { '.html': 'text/html', '.js': 'text/javascript', '.woff2': 'font/woff2' }[ext] || 'application/octet-stream' });
  fs.createReadStream(p).pipe(r);
}).listen(Number(process.env.PORT||8765));
(async () => {
  const b = await chromium.launch({ args: ['--use-gl=angle', '--use-angle=swiftshader', '--enable-unsafe-swiftshader', '--ignore-gpu-blocklist'] });
  const pg = await b.newPage({ viewport: { width: 1080, height: 1920 }, deviceScaleFactor: Number(process.env.DPR || 1.25) });
  pg.on('pageerror', e => console.log('ERR', e.message));
  pg.on('console', m => { const s = m.text(); if (m.type() === 'error' || s.startsWith('[log]')) console.log(m.type(), s.slice(0, 600)); });
  await pg.goto(`http://localhost:${process.env.PORT||8765}/${process.env.PAGE||'film.html'}${process.env.Q ? '?' + process.env.Q : ''}`);
  await pg.waitForFunction(() => window.renderAt && window.ready === true, null, { timeout: 300000 });
  const mode = process.argv[2], SPEED = Number(process.env.SPEED||1);
  if (mode === 'stills') {
    for (const t of process.argv[3].split(',').map(Number)) {
      await pg.evaluate(t => window.renderAt(t), t);
      await pg.screenshot({ path: `fs_${t}.jpg`, type: 'jpeg', quality: 92 });
    }
  } else {
    const out = process.argv[3], t0s = Number(process.argv[4]), t1s = Number(process.argv[5]), fps = 30, n = Math.round((t1s - t0s) * fps / SPEED);
    const tmp = out.replace(/\.mp4$/, '.tmp.mp4');
    const ff = spawn('ffmpeg', ['-y','-v','error','-f','image2pipe','-framerate',String(fps),'-c:v','mjpeg','-i','-','-vf','scale=1080:1920:flags=lanczos','-c:v','libx264','-preset','slow','-crf','16','-pix_fmt','yuv420p','-movflags','+faststart',tmp], { stdio: ['pipe','inherit','inherit'] });
    const st = Date.now();
    for (let i = 0; i < n; i++) {
      await pg.evaluate(t => window.renderAt(t), t0s + i * SPEED / fps);
      const buf = await pg.screenshot({ type: 'jpeg', quality: 92 });
      if (!ff.stdin.write(buf)) await new Promise(r => ff.stdin.once('drain', r));
      if (i % 30 === 0) console.log(`[log] ${out} frame ${i}/${n} avg ${((Date.now() - st) / 1000 / (i + 1)).toFixed(2)}s`);
    }
    ff.stdin.end();
    const code = await new Promise(r => ff.on('close', r));
    if (code === 0) fs.renameSync(tmp, out); else console.log('ffmpeg failed', code);
  }
  await b.close(); srv.close();
})();
```

## runall.sh
```bash
#!/bin/bash
# PORT=8801 ./runall.sh 0 2 4 6   (shot indices) — S must list every shot start plus the final end time, matching SH in film.html
# Never write a dollar sign followed by a digit in this skill: skill arguments get substituted into those and break the script.
cd "$(dirname "${BASH_SOURCE[0]}")"
S=(0 5.5 12 18 25.5 33 40.5 47 54 60)
for i in "$@"; do DPR=${DPR:-1.25} SPEED=${SPEED:-1.25} PORT=${PORT:-8801} PAGE=${PAGE:-film.html} node render.js video shot$i.mp4 ${S[$i]} ${S[$((i+1))]} >> job_${PORT:-8801}.log 2>&1; done
```
Start the two jobs with `(PORT=8801 nohup ./runall.sh 0 2 4 6 8 >/dev/null 2>&1 &); (PORT=8802 nohup ./runall.sh 1 3 5 7 >/dev/null 2>&1 &)`, then check from the foreground with calls like `sleep 540; ls shot*.mp4; grep -hF '[log]' job_*.log | tail -n 2` (each call can wait up to 10 minutes). To finish a heavy shot faster, render it as two halves in parallel, e.g. `DPR=1.25 SPEED=1.25 PORT=8823 nohup node render.js video shot7a.mp4 46.5 50 >job_a.log 2>&1 &` and the same for `shot7b.mp4 50 53.5` on another port. Concat list: `file 'shot0.mp4'` … one line per file, halves in order.

## post.js (renderer, environment maps, bloom, grade, depth of field)
```js
import * as THREE from 'three';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';
import { ShaderPass } from 'three/addons/postprocessing/ShaderPass.js';
import { BokehPass } from 'three/addons/postprocessing/BokehPass.js';
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';
const GradeShader = {
  uniforms: { tDiffuse: { value: null }, vig: { value: .32 }, sat: { value: 1.08 }, warm: { value: .04 } },
  vertexShader: `varying vec2 vUv; void main(){ vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0); }`,
  fragmentShader: `uniform sampler2D tDiffuse; uniform float vig, sat, warm; varying vec2 vUv;
    void main(){
      vec4 c = texture2D(tDiffuse, vUv);
      float l = dot(c.rgb, vec3(.2126,.7152,.0722));
      c.rgb = mix(vec3(l), c.rgb, sat);
      float k = smoothstep(0.0, 1.2, l);
      c.rgb *= mix(vec3(1.0-warm*.6, 1.0, 1.0+warm), vec3(1.0+warm, 1.0, 1.0-warm*.8), k);
      vec2 d = (vUv - .5) * vec2(1.0, 1.25);
      c.rgb *= 1.0 - vig * smoothstep(.25, .95, dot(d,d) * 2.2);
      gl_FragColor = c;
    }`
};
export function makeRenderer(W, H) {
  const dpr = window.devicePixelRatio || 1;   // set by render.js (DPR env); ffmpeg downscales with lanczos
  const renderer = new THREE.WebGLRenderer({ antialias: false, preserveDrawingBuffer: true, powerPreference: 'high-performance' });
  renderer.setPixelRatio(dpr); renderer.setSize(W, H);
  renderer.shadowMap.enabled = true; renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.outputColorSpace = THREE.SRGBColorSpace; renderer.toneMapping = THREE.ACESFilmicToneMapping;
  document.body.prepend(renderer.domElement);
  return { renderer, dpr };
}
const pmrem = (renderer, fn) => { const pm = new THREE.PMREMGenerator(renderer); const t = fn(pm).texture; pm.dispose(); return t };
export const roomEnv = renderer => pmrem(renderer, pm => pm.fromScene(new RoomEnvironment(renderer), .04));
export function equirectEnv(renderer, draw) {   // paint a 1024x512 panorama (sky gradient, bright window bands) for reflections
  const c = document.createElement('canvas'); c.width = 1024; c.height = 512; draw(c.getContext('2d'), 1024, 512);
  const tex = new THREE.CanvasTexture(c); tex.mapping = THREE.EquirectangularReflectionMapping; tex.colorSpace = THREE.SRGBColorSpace;
  return pmrem(renderer, pm => pm.fromEquirectangular(tex));
}
export async function exrEnv(renderer, url) {   // CC0 HDRIs from @pmndrs/assets, e.g. 'assets/night.exr'
  const { EXRLoader } = await import('three/addons/loaders/EXRLoader.js');
  const tex = await new EXRLoader().loadAsync(url); tex.mapping = THREE.EquirectangularReflectionMapping;
  return pmrem(renderer, pm => pm.fromEquirectangular(tex));
}
export function makePost(renderer, scene, camera, W, H, dpr, samples = 0) {
  const rt = new THREE.WebGLRenderTarget(W * dpr, H * dpr, { type: THREE.HalfFloatType, samples });
  const composer = new EffectComposer(renderer, rt); composer.setPixelRatio(dpr); composer.setSize(W, H);
  composer.addPass(new RenderPass(scene, camera));
  const bokeh = new BokehPass(scene, camera, { focus: 3, aperture: .02, maxblur: .022 }); bokeh.enabled = false; composer.addPass(bokeh);
  const bloom = new UnrealBloomPass(new THREE.Vector2(W, H), .38, .55, .92); composer.addPass(bloom);
  bloom.setSize(W * dpr / 2, H * dpr / 2);   // half-res bloom: same look, much cheaper
  const grade = new ShaderPass(GradeShader); composer.addPass(grade);
  composer.addPass(new OutputPass());
  return { composer, bloom, bokeh, grade };
}
```
In film.html, set `post.bloom.strength`, `renderer.toneMappingExposure` and `post.bokeh.enabled` every frame (the template does it from the shot's `bloom`, `exposure` and `dof`), with the bokeh focus at the camera-to-subject distance. Depth of field adds a full depth render, so keep it to one or two shots.

## lib.js (helpers for hero models and sets)
```js
import * as THREE from 'three';
export { THREE };
export const clamp = (x, a, b) => Math.max(a, Math.min(b, x));
export const lerp = (a, b, k) => a + (b - a) * k;
export const ease = x => { x = clamp(x, 0, 1); return x < .5 ? 2 * x * x : 1 - Math.pow(-2 * x + 2, 2) / 2 };
export const easeOut = x => { x = clamp(x, 0, 1); return 1 - Math.pow(1 - x, 3) };
export const smooth = (a, b, x) => { const t = clamp((x - a) / (b - a), 0, 1); return t * t * (3 - 2 * t) };
export const springy = (x, f = 2.2, d = 5) => x <= 0 ? 0 : 1 - Math.exp(-d * x) * Math.cos(f * Math.PI * x);   // 0 -> 1 with overshoot
export function rng(seed) { let s = (Math.abs(Math.floor(seed)) % 2147483646) + 1; return () => { s = (s * 16807) % 2147483647; return (s - 1) / 2147483646 } }
export const hexA = (hex, a) => { const n = parseInt(hex.slice(1), 16); return `rgba(${n >> 16 & 255},${n >> 8 & 255},${n & 255},${a})` };
export function canvasTex(w, h, draw, { repeat = [1, 1], srgb = true, aniso = 8, wrap = true } = {}) {
  const c = document.createElement('canvas'); c.width = w; c.height = h; draw(c.getContext('2d'), w, h);
  const t = new THREE.CanvasTexture(c); if (srgb) t.colorSpace = THREE.SRGBColorSpace; t.anisotropy = aniso;
  if (wrap) t.wrapS = t.wrapT = THREE.RepeatWrapping; t.repeat.set(repeat[0], repeat[1]); return t;
}
// draw something 9 times (offset by the canvas size) so the canvas tiles seamlessly
export const wrap9 = (W, H, fn) => { for (const dx of [-W, 0, W]) for (const dy of [-H, 0, H]) fn(dx, dy) };
const cr = (p0, p1, p2, p3, t) => { const t2 = t * t, t3 = t2 * t; return .5 * ((2 * p1) + (-p0 + p2) * t + (2 * p0 - 5 * p1 + 4 * p2 - p3) * t2 + (-p0 + 3 * p1 - 3 * p2 + p3) * t3) };
// control rings [y, rx, rz, cz] -> dense smooth rings (Catmull-Rom)
export function smoothRings(ctrl, per = 4) {
  const n = ctrl.length, out = [], g = (i, k) => ctrl[clamp(i, 0, n - 1)][k];
  for (let i = 0; i < n - 1; i++) for (let s = 0; s < per; s++) { const o = []; for (let k = 0; k < 4; k++) o.push(cr(g(i - 1, k), g(i, k), g(i + 1, k), g(i + 2, k), s / per)); out.push(o) }
  out.push([...ctrl[n - 1]]); return out;
}
export function fixSeam(g, segs, nRings) {   // average normals across the u=0/1 seam
  const nr = g.attributes.normal;
  for (let i = 0; i < nRings; i++) { const a = i * (segs + 1), b = a + segs, x = nr.getX(a) + nr.getX(b), y = nr.getY(a) + nr.getY(b), z = nr.getZ(a) + nr.getZ(b), l = Math.hypot(x, y, z) || 1; nr.setXYZ(a, x / l, y / l, z / l); nr.setXYZ(b, x / l, y / l, z / l) }
  nr.needsUpdate = true;
}
// tube of elliptical rings, bottom -> top. uv: u around (0..1, front at u=0), v = arc length in metres
export function loft(rings, segs = 64) {
  const n = rings.length, pos = new Float32Array(n * (segs + 1) * 3), uv = new Float32Array(n * (segs + 1) * 2), idx = [], vs = [0];
  for (let i = 1; i < n; i++) { const a = rings[i - 1], b = rings[i]; vs.push(vs[i - 1] + Math.hypot(b[0] - a[0], ((b[1] + b[2]) - (a[1] + a[2])) / 2)) }
  for (let i = 0; i < n; i++) for (let j = 0; j <= segs; j++) {
    const [y, rx, rz, cz] = rings[i], th = j / segs * Math.PI * 2, k = i * (segs + 1) + j;
    pos.set([Math.sin(th) * rx, y, (cz || 0) + Math.cos(th) * rz], k * 3); uv.set([j / segs, vs[i]], k * 2);
  }
  for (let i = 0; i < n - 1; i++) for (let j = 0; j < segs; j++) { const a = i * (segs + 1) + j, b = a + 1, c = a + segs + 1, d = c + 1; idx.push(a, b, c, b, d, c) }
  const g = new THREE.BufferGeometry(); g.setAttribute('position', new THREE.BufferAttribute(pos, 3)); g.setAttribute('uv', new THREE.BufferAttribute(uv, 2)); g.setIndex(idx);
  g.computeVertexNormals(); fixSeam(g, segs, n); g.userData = { segs, n }; return g;
}
export function ellipseCurve(y, rx, rz, cz = 0, n = 48) {   // closed ring for TubeGeometry trims
  const pts = []; for (let i = 0; i < n; i++) { const a = i / n * Math.PI * 2; pts.push(new THREE.Vector3(Math.sin(a) * rx, y, cz + Math.cos(a) * rz)) }
  return new THREE.CatmullRomCurve3(pts, true);
}
// colour change as a downward sweep with a gold glow band. The material needs a map. Set m.userData.u.map2.value to the
// new texture and move m.userData.u.wipeY.value from above the character (10 = not started) to below the feet (-1 = done).
export function addWipe(m, glow = new THREE.Color(1, .78, .35)) {
  const u = { map2: { value: m.map }, wipeY: { value: 10 }, wipeW: { value: .035 }, glowC: { value: glow } };
  m.userData.u = u; m.customProgramCacheKey = () => 'wipe1';
  m.onBeforeCompile = sh => {
    Object.assign(sh.uniforms, u);
    sh.vertexShader = sh.vertexShader.replace('#include <common>', `#include <common>
varying float vWY;`).replace('#include <begin_vertex>', `#include <begin_vertex>
vWY = (modelMatrix * vec4(transformed, 1.0)).y;`);
    sh.fragmentShader = sh.fragmentShader.replace('#include <common>', `#include <common>
varying float vWY; uniform sampler2D map2; uniform float wipeY, wipeW; uniform vec3 glowC;`)
      .replace('#include <map_fragment>', `#ifdef USE_MAP
  diffuseColor *= mix(texture2D(map, vMapUv), texture2D(map2, vMapUv), smoothstep(wipeY - wipeW, wipeY + wipeW, vWY));
#endif`)
      .replace('#include <emissivemap_fragment>', `#include <emissivemap_fragment>
totalEmissiveRadiance += glowC * 4.0 * (1.0 - smoothstep(0.0, wipeW * 1.6, abs(vWY - wipeY)));`);
  };
  return m;
}
```

## music.py (generated soundtrack + SFX; times are TIMELINE seconds)
```python
import numpy as np, wave
SP=1.25            # must match SPEED
TIMELINE_END=60.0  # last shot end
NIGHT_AT=54.0      # start of the night/return shot (music bed stops here); None to run to the end
MUTE=[]            # [(start,end)] timeline windows where the bed drops out (a standoff, a hush)
# EVENTS: (kind, timeline_time, extra). Kinds: poof, pips (extra=count), qpops (count), whoosh (duration), boing, thud, chime, clicks (count), return
EVENTS=[('poof',9.1,None),('poof',21.4,None),('pips',22.4,13),('qpops',12.8,8),('whoosh',34.2,5.5),('return',54.0,None)]
SR=44100; DUR=TIMELINE_END/SP; N=int(SR*DUR); out=np.zeros((N,2)); rng=np.random.default_rng(3)
def note(f): return 440*2**((f-69)/12)
def add(sig,t0,pan=0.0,gain=1.0):
    i=int(t0/SP*SR); j=min(N,i+len(sig))
    if i>=N: return
    s=sig[:j-i]*gain; out[i:j,0]+=s*(1-pan)*.53; out[i:j,1]+=s*(1+pan)*.53
def pluck(m,dur=1.2,bright=1.0):
    t=np.arange(int(SR*dur))/SR; f=note(m)
    s=np.sin(2*np.pi*f*t)+.35*bright*np.sin(4*np.pi*f*t)*np.exp(-t*6)+.15*np.sin(6*np.pi*f*t)*np.exp(-t*9)
    return s*np.exp(-t*3.2)*np.minimum(1,t*400)
def pad(ms,dur):
    t=np.arange(int(SR*dur))/SR; s=np.zeros_like(t)
    for m in ms:
        for d in (-.12,.12): s+=np.sin(2*np.pi*note(m)*(1+d/100)*t)
    return s*np.minimum(1,t/.6)*np.minimum(1,(dur-t)/.6)/len(ms)/2
def bass(m,dur):
    t=np.arange(int(SR*dur))/SR; f=note(m)
    return (np.sin(2*np.pi*f*t)+.2*np.sin(4*np.pi*f*t))*np.exp(-t*1.5)*np.minimum(1,t*200)
def noise(dur,lp):
    n=rng.standard_normal(int(SR*dur)); y=np.zeros_like(n)
    for i in range(1,len(n)): y[i]=y[i-1]+lp*(n[i]-y[i-1])
    return y/np.max(np.abs(y))
beat=60/84; bar=4*beat   # timeline units; real tempo = 84*SP bpm
prog=[(57,[57,60,64]),(53,[53,57,60]),(48,[52,55,60]),(55,[55,59,62])]  # Am F C G music-box
arp=[0,1,2,1,0,2,1,2]; end=NIGHT_AT or TIMELINE_END; t=0.0; k=0
muted=lambda x:any(a<=x<b for a,b in MUTE)
while t<end-0.01:
    root,ch=prog[k%4]
    if not muted(t): add(pad([c+12 for c in ch],bar/SP+.3),t,0,.16); add(bass(root-12,bar/SP),t,0,.35)
    for i in range(8):
        tt=t+i*beat/2
        if tt>=end: break
        if not muted(tt): add(pluck(ch[arp[i]]+24,1.0),tt,(-.3 if i%2 else .3),.22 if t>5 else .15)
    if k%2==1 and t>5:
        for i,m in enumerate([ch[2],ch[1],ch[0]]):
            if not muted(t+i*beat*1.33): add(pluck(m+36,1.4,.6),t+i*beat*1.33,0,.10)
    t+=bar; k+=1
def pip(t0,f): tt=np.arange(int(SR*.12))/SR; add(np.sin(2*np.pi*f*tt)*np.exp(-tt*30),t0,rng.uniform(-.5,.5),.15)
def T(d): return np.arange(int(SR*d))/SR
def bell(m,dur=2.4):   # PA-chime style bell
    t=T(dur); f=note(m); s=0
    for r,a,d in ((1,1,1.6),(2.0,.5,2.4),(2.76,.3,3.5),(5.4,.14,5)): s=s+a*np.sin(2*np.pi*f*r*t)*np.exp(-t*d)
    return s*np.minimum(1,t*600)
def boing(t0,f0=150,f1=330,g=.5):   # springy growth or pop
    t=T(1.0); f=(f0+(f1-f0)*(1-np.exp(-6*t)))*(1+.22*np.exp(-4*t)*np.sin(2*np.pi*13*t))
    ph=2*np.pi*np.cumsum(f)/SR; add((np.sin(ph)+.3*np.sin(2*ph))*np.exp(-t*3)*np.minimum(1,t*300),t0,0,g)
def thud(t0,g=.6):   # impact or kick drum
    t=T(.35); ph=2*np.pi*np.cumsum(95*np.exp(-t*5)+45)/SR; add(np.sin(ph)*np.exp(-t*9),t0,0,g)
def click(t0,g=.2,pan=0):   # ratchet, tape measure, camera shutter
    n=np.diff(rng.standard_normal(int(SR*.012)+1)); t=np.arange(len(n))/SR; add(n*np.exp(-t*400),t0,pan,g)
for kind,t0,x in EVENTS:
    if kind=='poof':
        n=noise(1.0,.08); tt=np.arange(len(n))/SR; add(n*np.minimum(1,tt*60)*np.exp(-tt*4.5),t0,0,.9)
        tt=np.arange(int(SR*.18))/SR; add(np.sin(2*np.pi*(900-3000*tt)*tt)*np.exp(-tt*18),t0,0,.25)
    elif kind=='pips':
        for i in range(x): pip(t0+i*.04,1300+i*40)
    elif kind=='qpops':
        for i in range(x): pip(t0+i*.22,700+((i*7)%5)*90)
    elif kind=='whoosh':
        n=noise(x,.03); tt=np.arange(len(n))/SR; add(n*np.sin(np.pi*tt/x)**2,t0,0,.35)
    elif kind=='boing': boing(t0)
    elif kind=='thud': thud(t0)
    elif kind=='chime': add(bell(76),t0,-.1,.22); add(bell(72),t0+.5,.1,.22)
    elif kind=='clicks':
        for i in range(x): click(t0+i*.045)
    elif kind=='return':
        tt=np.arange(int(SR*1.2))/SR; add(np.sin(2*np.pi*55*tt)*np.exp(-tt*2),t0,0,.4)
        for i,m in enumerate([72,76,79,84,88,91]): add(pluck(m,1.6),t0+1.2+i*.07,(i%2-.5),.18)
        add(pad([60,64,67,72],4.5),t0+1.3,0,.3)
        c=noise(3.0,.25); tt=np.arange(len(c))/SR; add(c*np.minimum(1,tt/.4)*np.exp(-tt*.6)*.3,t0+1.8,0,1)
rms=np.sqrt(np.mean(out**2)); out*=.13/rms; out=np.tanh(out*1.3)/np.tanh(1.3)*.94   # RMS + soft limit; peak-normalising lets one loud SFX make the whole track quiet
fl=int(SR*1.5); out[-fl:]*=np.linspace(1,0,fl)[:,None]
with wave.open('music.wav','wb') as w: w.setnchannels(2);w.setsampwidth(2);w.setframerate(SR);w.writeframes((out*32767).astype(np.int16).tobytes())
```

## film.html: reference template (the Somerset / Yo-Chi episode)
This is a complete working episode. For a new story, copy it and change:
- the title HTML and the location pill
- the set (only if the location differs)
- `shop(...)` names and colours, and which shops get puffs
- the casts and the `SH` shot array
- `runall.sh` S times and `music.py` EVENTS to match

Keep the overlay CSS, the person/pose system, and the `renderAt` contract (`window.renderAt(T)`, `window.ready = true`). For bigger episodes, split the code into ES modules next to film.html (the importmap maps `three/addons/`): `lib.js`, `post.js`, `people.js` (hero models plus the blocky crowd), and `sets.js` (one function per set, each in its own group at its own origin, returning its lights and the materials a shot may change). Keep film.html for the cast, the `SH` array and `renderAt`. Shots can add `bloom`, `exposure`, `dof`, and `fov`/`focus` from `cam(t)`. Each shot sets its set's mood (lights, emissives, sky, fog) itself, so frames stay independent of render order.

```html
<!doctype html>
<html><head><meta charset="utf-8">
<style>
@font-face{font-family:Crimson;src:url(node_modules/@fontsource/crimson-pro/files/crimson-pro-latin-400-normal.woff2);font-weight:400;font-style:normal}
@font-face{font-family:Crimson;src:url(node_modules/@fontsource/crimson-pro/files/crimson-pro-latin-400-italic.woff2);font-weight:400;font-style:italic}
@font-face{font-family:Inter;src:url(node_modules/@fontsource/inter/files/inter-latin-500-normal.woff2);font-weight:500}
@font-face{font-family:Inter;src:url(node_modules/@fontsource/inter/files/inter-latin-700-normal.woff2);font-weight:700}
html,body{margin:0;background:#000;width:1080px;height:1920px;overflow:hidden}
canvas{display:block}
#ui{position:absolute;inset:0;pointer-events:none;color:#fff}
#title{position:absolute;top:440px;left:50px;right:50px;text-align:center;font:400 84px/1.12 Crimson;text-shadow:0 4px 24px rgba(0,0,0,.6)}
#title b{font-weight:400;color:#ffd35a}
#hud{position:absolute;top:590px;left:56px;text-shadow:0 2px 10px rgba(0,0,0,.6)}
#hud .k{font:700 22px Inter;letter-spacing:.12em}
#hud .v{font:400 62px/1.05 Crimson;margin-top:4px}
#hud .m{font:700 20px Inter;letter-spacing:.14em;opacity:.95;margin-top:6px}
#pill{display:inline-block;margin-top:40px;font:700 17px Inter;letter-spacing:.14em;padding:9px 18px;border-radius:30px;background:rgba(0,0,0,.45);border:1.5px solid rgba(255,255,255,.75)}
#cap{position:absolute;top:1230px;left:70px;right:70px;text-align:center;font:italic 400 60px/1.2 Crimson;text-shadow:0 3px 18px rgba(0,0,0,.8)}
#hudbg{position:absolute;left:0;top:470px;width:820px;height:520px;opacity:0;background:radial-gradient(ellipse at 18% 48%,rgba(0,0,0,.34),rgba(0,0,0,0) 68%)}
#capbg{position:absolute;left:0;right:0;top:1150px;height:330px;opacity:0;background:radial-gradient(ellipse at 50% 50%,rgba(0,0,0,.3),rgba(0,0,0,0) 70%)}
#black{position:absolute;inset:0;background:#000;opacity:0}
#end{position:absolute;top:900px;left:0;right:0;text-align:center;font:italic 400 64px Crimson;color:#fff;opacity:0}
</style></head>
<body>
<div id="ui">
  <div id="hudbg"></div><div id="capbg"></div>
  <div id="title">What if <b>Yo-Chi</b> and<br><b>Molly Tea</b> just…<br>disappeared?</div>
  <div id="hud"><div class="k" id="hudk"></div><div class="v" id="hudv"></div><div class="m" id="hudm"></div><div id="pill">▶ SOMERSET, SINGAPORE</div></div>
  <div id="cap"></div>
  <div id="black"></div><div id="end">Part 2?</div>
</div>
<script type="importmap">{"imports":{"three":"./node_modules/three/build/three.module.js","three/addons/":"./node_modules/three/examples/jsm/"}}</script>
<script type="module">
import * as THREE from 'three';
import {makeRenderer,makePost} from './post.js';
await Promise.all(['700 40px Inter','500 40px Inter','400 40px Crimson','italic 400 40px Crimson'].map(f=>document.fonts.load(f)));
const W=1080,H=1920;
let seed=11;const rnd=()=>{seed=(seed*16807)%2147483647;return (seed-1)/2147483646};
const R=(a,b)=>a+(b-a)*rnd();const pick=a=>a[Math.floor(rnd()*a.length)];
const clamp=(x,a,b)=>Math.max(a,Math.min(b,x));
const ease=x=>{x=clamp(x,0,1);return x<.5?2*x*x:1-Math.pow(-2*x+2,2)/2};
const lerp=(a,b,k)=>a+(b-a)*k;
const V=(x,y,z)=>new THREE.Vector3(x,y,z);

const {renderer,dpr}=makeRenderer(W,H);renderer.toneMappingExposure=1.05;   // supersampled via render.js DPR
const scene=new THREE.Scene();
const camera=new THREE.PerspectiveCamera(58,W/H,0.1,400);
const post=makePost(renderer,scene,camera,W,H,dpr);   // bloom + grade, optional depth of field

function skyTex(stops){const c=document.createElement('canvas');c.width=4;c.height=256;const g=c.getContext('2d');const gr=g.createLinearGradient(0,0,0,256);stops.forEach(([o,col])=>gr.addColorStop(o,col));g.fillStyle=gr;g.fillRect(0,0,4,256);const t=new THREE.CanvasTexture(c);t.colorSpace=THREE.SRGBColorSpace;return t}
const SKY_DUSK=skyTex([[0,'#2b3f7a'],[.55,'#6d78b8'],[1,'#e7a98f']]);
const SKY_NIGHT=skyTex([[0,'#05070f'],[.6,'#111a35'],[1,'#2a2348']]);
scene.fog=new THREE.Fog(0x8a86b4,60,170);

const mat=(c,o={})=>new THREE.MeshStandardMaterial(Object.assign({color:c,roughness:.85,metalness:0},o));
function box(w,h,d,m,x,y,z,parent=scene,cast=true){const me=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),m);me.position.set(x,y,z);me.castShadow=cast;me.receiveShadow=true;parent.add(me);return me}
function label(text,{w=1024,h=256,bg='#111',fg='#fff',font='700 150px Inter',sub=null,subFont='500 52px Inter'}={}){
  const c=document.createElement('canvas');c.width=w;c.height=h;const g=c.getContext('2d');
  if(bg){g.fillStyle=bg;g.fillRect(0,0,w,h)}g.fillStyle=fg;g.textAlign='center';g.textBaseline='middle';
  g.font=font;g.fillText(text,w/2,sub?h*.42:h/2);
  if(sub){g.font=subFont;g.globalAlpha=.85;g.fillText(sub,w/2,h*.8)}
  const t=new THREE.CanvasTexture(c);t.colorSpace=THREE.SRGBColorSpace;t.anisotropy=8;return t}

// lights
const hemi=new THREE.HemisphereLight(0xbfc8ff,0x6b5a4a,1.1);scene.add(hemi);
const sun=new THREE.DirectionalLight(0xffe2c4,1.6);sun.position.set(30,40,10);sun.castShadow=true;
sun.shadow.mapSize.set(2048,2048);Object.assign(sun.shadow.camera,{left:-40,right:40,top:40,bottom:-40,near:1,far:120});sun.shadow.bias=-0.0005;
sun.target.position.set(0,0,-8);scene.add(sun,sun.target);
const warm=new THREE.PointLight(0xffc488,40,30,1.6);warm.position.set(-6,5,-6);scene.add(warm);
const warm2=new THREE.PointLight(0xffc488,40,30,1.6);warm2.position.set(-6,5,-24);scene.add(warm2);
const glowL=new THREE.PointLight(0xffe9b0,0,25,1.4);glowL.position.set(-7,3,-6);scene.add(glowL);

// floor
{const c=document.createElement('canvas');c.width=c.height=512;const g=c.getContext('2d');
for(let i=0;i<8;i++)for(let j=0;j<4;j++){const v=Math.floor(R(128,160));g.fillStyle=`rgb(${v},${v-2},${v+4})`;g.fillRect(i*64,j*128,64,128);}
g.strokeStyle='rgba(60,60,70,.35)';g.lineWidth=2;for(let i=0;i<=8;i++){g.beginPath();g.moveTo(i*64,0);g.lineTo(i*64,512);g.stroke()}for(let j=0;j<=4;j++){g.beginPath();g.moveTo(0,j*128);g.lineTo(512,j*128);g.stroke()}
const t=new THREE.CanvasTexture(c);t.wrapS=t.wrapT=THREE.RepeatWrapping;t.repeat.set(5,24);t.colorSpace=THREE.SRGBColorSpace;
const f=new THREE.Mesh(new THREE.PlaneGeometry(20,96),mat(0xffffff,{map:t,roughness:.55}));f.rotation.x=-Math.PI/2;f.position.set(0,0,-20);f.receiveShadow=true;scene.add(f);}

// left wall + canopy
box(1,14,96,mat(0xd9d4cc),-10.5,7,-20);
const slatCols=[0xc0392b,0xe67e22,0xf2efe9,0xa0703f,0xd35400];
box(11,.4,70,mat(0x3a3a40),-4.5,7.4,-14,scene,false);
for(let z=20;z>-48;z-=0.55)box(10.5,.18,.16,mat(pick(slatCols),{roughness:.6}),-4.6,7.0,z,scene,false);
const stripMat=new THREE.MeshBasicMaterial({color:0xfff6e0});
for(let z=20;z>-48;z-=3.3)box(10.5,.05,.08,stripMat,-4.6,6.88,z,scene,false);

function shop(z,len,name,sub,signBg,signFg,inside){
  const g=new THREE.Group();g.position.set(-9.9,0,z);scene.add(g);
  box(.3,4.2,len,mat(0x2c2c30),0,2.1,0,g);
  const glass=new THREE.Mesh(new THREE.PlaneGeometry(len-.6,3.6),new THREE.MeshStandardMaterial({color:inside,emissive:inside,emissiveIntensity:.55,roughness:.2}));
  glass.rotation.y=Math.PI/2;glass.position.set(.2,1.9,0);g.add(glass);
  const dark=mat(0x111);const s=new THREE.Mesh(new THREE.BoxGeometry(.25,1.3,len),[new THREE.MeshStandardMaterial({map:label(name,{w:1536,h:256,bg:signBg,fg:signFg,font:'700 140px Inter',sub,subFont:'500 46px Inter'}),emissive:0xffffff,emissiveIntensity:.35}),dark,dark,dark,dark,dark]);
  s.position.set(.3,4.9,0);g.add(s);
  for(let i=0;i<6;i++)box(R(.5,1.1),R(.6,1.1),R(.6,1.3),mat(pick([0xf3e3c3,0xffffff,0xe8c39e,0xbfe3c0])),.7,R(.4,.7),R(-len/2+1,len/2-1),g);
  const ghost=new THREE.Mesh(new THREE.PlaneGeometry(len,5.6),new THREE.MeshStandardMaterial({color:0xece8e1,roughness:.9}));
  ghost.rotation.y=Math.PI/2;ghost.position.set(-9.99,2.8,z);ghost.visible=false;scene.add(ghost);
  return {g,ghost,z}}
shop(5,8,'coffeesmyth','','#2a2a2a','#f5f5f5',0xffe7c4);
const YOCHI=shop(-6,9,'YO-CHI','frozen yogurt & sadness','#1a1a1a','#ffffff',0xfff1d6);
const MOLLY=shop(-17,8,'MOLLY TEA','jasmine milk tea • 1-for-1','#1f5a3c','#f3ffe9',0xe8ffe9);
shop(-28,9,'SOMERSET','such shopping','#5b2130','#fff',0xffe3e3);

// right side
function muralTex(){const c=document.createElement('canvas');c.width=128;c.height=512;const g=c.getContext('2d');g.fillStyle='#f4efe6';g.fillRect(0,0,128,512);
 for(let i=0;i<26;i++){g.fillStyle=pick(['#e74c3c','#f1c40f','#2ecc71','#3498db','#9b59b6','#ff8fab','#1abc9c']);g.globalAlpha=.75;g.beginPath();g.ellipse(R(0,128),R(0,512),R(10,40),R(20,70),R(0,3),0,7);g.fill()}
 const t=new THREE.CanvasTexture(c);t.colorSpace=THREE.SRGBColorSpace;return t}
box(1,14,96,mat(0xcfd2d6),10.5,7,-20);
for(let z=20;z>-44;z-=5)box(.9,6.5,.9,mat(0xffffff,{map:muralTex()}),8.9,3.25,z);
box(4,.35,96,mat(0xbfc3c8),8.3,6.7,-20);
for(let z=20;z>-44;z-=1.6)box(.04,1,.04,mat(0x8899aa),6.4,7.4,z,scene,false);
{const rail=new THREE.Mesh(new THREE.BoxGeometry(.04,1,64),new THREE.MeshStandardMaterial({color:0x99ccdd,transparent:true,opacity:.35}));rail.position.set(6.4,7.4,-12);scene.add(rail);}
box(1.6,.5,9,mat(0x2bb5a0,{roughness:.5}),6.9,.25,1);{const b2=box(1.6,.5,5,mat(0x2bb5a0,{roughness:.5}),6.4,.25,-5.6);b2.rotation.y=.25}
{const c=document.createElement('canvas');c.width=256;c.height=768;const g=c.getContext('2d');g.fillStyle='#151515';g.fillRect(0,0,256,768);g.fillStyle='#fff';g.font='700 40px Inter';g.textAlign='center';
 g.fillText('DISCOVER',128,90);g.fillText('MORE ↑',128,140);g.font='500 30px Inter';g.fillText('(dessert not',128,420);g.fillText('included)',128,460);
 const tt=new THREE.CanvasTexture(c);tt.colorSpace=THREE.SRGBColorSpace;
 const ban=new THREE.Mesh(new THREE.PlaneGeometry(1.3,3.9),new THREE.MeshStandardMaterial({map:tt,side:THREE.DoubleSide,emissive:0xffffff,emissiveMap:tt,emissiveIntensity:.25}));ban.position.set(5.6,4.4,-12);ban.rotation.y=-.35;scene.add(ban);}
// far end LED + red column
const ledC=document.createElement('canvas');ledC.width=256;ledC.height=640;const ledG=ledC.getContext('2d');const ledT=new THREE.CanvasTexture(ledC);ledT.colorSpace=THREE.SRGBColorSpace;
function drawLED(t){ledG.fillStyle='#0b3d91';ledG.fillRect(0,0,256,640);
 for(let i=0;i<14;i++){const y=(i*53+t*60)%720-40;ledG.fillStyle=i%3?'#4fc3f7':'#e1f5fe';ledG.globalAlpha=.7;ledG.beginPath();ledG.ellipse(128+Math.sin(t*1.3+i)*80,y,60+20*Math.sin(i),26,Math.sin(t+i)*.6,0,7);ledG.fill()}
 ledG.globalAlpha=1;ledT.needsUpdate=true}
const led=new THREE.Mesh(new THREE.PlaneGeometry(7,17.5),new THREE.MeshBasicMaterial({map:ledT}));led.position.set(-1,9.5,-47);scene.add(led);
box(1.2,19,1.2,mat(0xb03a2e),3.4,9.5,-46);box(30,22,2,mat(0xd8d5d0),-6,11,-49);
for(let i=0;i<10;i++)box(R(8,14),R(20,50),R(8,14),mat(pick([0x9aa3c8,0xb7a8c9,0x8e9bbf])),R(-30,30),15,R(-80,-60),scene,false);

// puff of smoke for vanishing
function makePuff(x,y,z,spread){const g=new THREE.Group();g.position.set(x,y,z);scene.add(g);const parts=[];
 for(let i=0;i<22;i++){const m=new THREE.Mesh(new THREE.SphereGeometry(1,14,10),new THREE.MeshStandardMaterial({color:0xf4f4f4,transparent:true,opacity:0,roughness:1,depthWrite:false}));
  const d=V(R(.2,1),R(-.8,1),R(-1,1)).normalize();m.userData={d,s:R(.6,1.3),sp:R(.7,1.3)*spread};g.add(m);parts.push(m)}
 return k=>{g.visible=k>0&&k<1;for(const m of parts){const u=m.userData;const e=1-Math.pow(1-clamp(k,0,1),3);m.position.copy(u.d).multiplyScalar(e*u.sp);m.scale.setScalar(u.s*(.3+e*1.1));m.material.opacity=.95*(1-clamp(k,0,1))}}}
const puffY=makePuff(-9.2,2.4,-6,4.5),puffM=makePuff(-9.2,2.4,-17,4.2);
function setShop(S,k,puff,tSince){ // k: 0 present ... 1 gone; tSince: puff progress 0..1 (negative = no puff)
  const sx=k<=0?1:Math.max(0,1-ease(k));S.g.scale.set(sx,1+(k>0?.25*Math.sin(Math.min(1,k)*Math.PI):0),sx);S.g.visible=sx>0.001;S.ghost.visible=k>0.4;
  puff(tSince);}

// cardboard stall
const stall=new THREE.Group();stall.position.set(-8.4,0,-5);scene.add(stall);
{const cb=mat(0xb98b55);box(1.0,1.1,2.2,cb,0,.55,0,stall);box(.04,1.4,.04,cb,.3,1.8,-.9,stall);box(.04,1.4,.04,cb,.3,1.8,.9,stall);
 const st=label('YO-CHI (REAL)',{w:1024,h:300,bg:'#c89a62',fg:'#222',font:'italic 130px Crimson',sub:'trust me bro',subFont:'italic 70px Crimson'});
 const sg=new THREE.Mesh(new THREE.PlaneGeometry(2.4,.72),new THREE.MeshStandardMaterial({map:st}));sg.rotation.y=Math.PI/2;sg.rotation.z=.04;sg.position.set(.32,2.35,0);stall.add(sg);
 const bowl=new THREE.Mesh(new THREE.SphereGeometry(.12,14,8,0,7,Math.PI/2,Math.PI/2),mat(0xffffff));bowl.position.set(.2,1.2,.4);stall.add(bowl);}
stall.visible=false;
// tumbleweed
const tumble=new THREE.Mesh(new THREE.IcosahedronGeometry(.45,1),new THREE.MeshStandardMaterial({color:0x9c7a45,wireframe:true}));tumble.castShadow=true;scene.add(tumble);tumble.visible=false;

// ---------- people ----------
const SKIN=[0xf1c9a5,0xe0ac7e,0xc68642,0xf5d6b8,0x8d5524];
const HAIR=[0x1a1a1a,0x2b1d14,0x4a3020,0x111111];
const STY={
 adult:{shirt:[0xe74c3c,0x2ecc71,0x3498db,0xf1c40f,0x9b59b6,0xffffff,0x1abc9c,0x34495e,0xe67e22,0xd0d0d0],pants:[0x2c3e50,0x3b3b3b,0x4a6fa5,0xd8cfc0,0x6b6b6b]},
 xmm:{shirt:[0xffb3c7,0xffffff,0xc9b3ff,0xb3e5ff,0x111111,0xf6d365,0xff8fab],pants:[0xe9e1d3,0x7da0c8,0x111111,0xd6c7b0,0x9bb7d4]},
 xdd:{shirt:[0x111111,0x1d1d1d,0xffffff,0x3b3b3b,0x6b5b4b,0x2b2b2b],pants:[0x111111,0x4a4436,0x2f3a2f,0x555555,0x22252b]}};
const all=[];
function person(style='adult',o={}){
  const S=STY[style];const shirt=o.shirt??pick(S.shirt),pants=o.pants??pick(S.pants),skin=o.skin??pick(SKIN),hair=o.hair??pick(HAIR);
  const long=style==='xmm'?true:style==='xdd'?false:rnd()<.5;
  const g=new THREE.Group();const body=new THREE.Group();g.add(body);
  const ms=mat(shirt),mp=mat(pants),mk=mat(skin),mh=mat(hair);
  if(style==='xmm'){box(.44,.34,.25,ms,0,1.31,0,body);box(.4,.12,.22,mk,0,1.08,0,body);box(.44,.16,.26,mp,0,.95,0,body);}
  else box(.46,.62,.26,ms,0,1.17,0,body);
  const headG=new THREE.Group();headG.position.set(0,1.5,0);body.add(headG);
  const head=new THREE.Mesh(new THREE.SphereGeometry(.15,18,14),mk);head.scale.set(1,1.15,1);head.position.y=.14;head.castShadow=true;headG.add(head);
  const cap=new THREE.Mesh(new THREE.SphereGeometry(.158,18,10,0,Math.PI*2,0,Math.PI*.55),mh);cap.position.y=.16;cap.rotation.x=-.25;cap.castShadow=true;headG.add(cap);
  if(long)box(.32,.46,.1,mh,0,-.02,-.11,headG);
  if(style==='xdd'){box(.3,.08,.08,mh,0,.24,.12,headG);box(.26,.16,.06,mh,0,.04,-.15,headG);
    if(rnd()<.4){const c=new THREE.Mesh(new THREE.CylinderGeometry(.165,.165,.12,16),mat(pick([0x111111,0xf0f0f0,0x2c3e50])));c.position.y=.27;headG.add(c);box(.26,.03,.18,c.material,0,.23,.2,headG);}}
  const limb=(w,l,m,x,y)=>{const p=new THREE.Group();p.position.set(x,y,0);body.add(p);box(w,l,w,m,0,-l/2,0,p);return p};
  const armMat=style==='xmm'&&rnd()<.6?mk:ms;
  const armL=limb(.12,.6,armMat,-.3,1.44),armR=limb(.12,.6,armMat,.3,1.44);
  [armL,armR].forEach(a=>box(.1,.1,.1,mk,0,-.62,0,a));
  const lw=style==='adult'?.16:.2;
  const mkLeg=x=>{const hip=new THREE.Group();hip.position.set(x,.86,0);body.add(hip);box(lw,.44,lw,mp,0,-.22,0,hip);const knee=new THREE.Group();knee.position.y=-.44;hip.add(knee);box(lw-.01,.42,lw-.01,mp,0,-.21,0,knee);box(.16,.08,.26,mat(style==='xdd'?pick([0xf0f0f0,0x111111,0xe0e0e0]):0xf0f0f0),0,-.44,.05,knee);return {hip,knee}};
  const legL=mkLeg(-.12),legR=mkLeg(.12);
  if(style==='xmm'&&rnd()<.6)box(.08,.32,.28,mat(pick([0xf3e6d0,0x111111,0xd9a7c7])),-.38,1.0,0,body);
  g.scale.setScalar(o.scale??(style==='adult'?R(.95,1.06):R(.9,.98)));scene.add(g);
  const q=new THREE.Sprite(new THREE.SpriteMaterial({map:label('?',{w:128,h:128,bg:null,fg:'#fff',font:'700 110px Inter'}),transparent:true,depthTest:false}));
  q.position.set(0,2.25,0);q.scale.set(0,0,0);g.add(q);
  const prop=new THREE.Group();prop.position.set(0,-.68,.04);armR.add(prop);
  const cup=new THREE.Group();prop.add(cup);
  {const c=new THREE.Mesh(new THREE.CylinderGeometry(.055,.045,.18,12),mat(0xe9dcc3,{roughness:.4}));c.position.y=.09;cup.add(c);
   const lid=new THREE.Mesh(new THREE.CylinderGeometry(.058,.058,.02,12),mat(0xffffff));lid.position.y=.185;cup.add(lid);
   box(.015,.14,.015,mat(0x2e7d32),0,.25,0,cup,false);const lbl=new THREE.Mesh(new THREE.CylinderGeometry(.056,.05,.06,12),mat(0x1f5a3c));lbl.position.y=.09;cup.add(lbl);}
  const bowl=new THREE.Mesh(new THREE.SphereGeometry(.15,14,8,0,7,Math.PI/2,Math.PI/2),mat(0xffffff,{roughness:.4,side:THREE.DoubleSide}));bowl.position.y=.12;prop.add(bowl);
  cup.visible=false;bowl.visible=false;
  const P={g,body,headG,armL,armR,legL,legR,q,prop,cup,bowl,phase:R(0,6.28),style};all.push(P);g.visible=false;return P;
}
function pose(p,{walk=0,t=0,speed=7.5,sit=false,armUp=0,headYaw=0,headPitch=0,slump=0,lie=false,cheer=0}){
  const ph=t*speed+p.phase,s=Math.sin(ph)*walk;
  p.g.rotation.x=0;
  p.legL.hip.rotation.x=sit?-1.5:s*.55;p.legR.hip.rotation.x=sit?-1.45:-s*.55;
  p.legL.knee.rotation.x=sit?1.5:Math.max(0,-Math.sin(ph+.6))*walk*.9;p.legR.knee.rotation.x=sit?1.45:Math.max(0,Math.sin(ph+.6))*walk*.9;
  p.armL.rotation.x=-s*.5-cheer*2.8;p.armR.rotation.x=s*.5-armUp-cheer*2.8;p.armL.rotation.z=.08+cheer*.3;p.armR.rotation.z=-.08-cheer*.3;
  p.prop.rotation.x=-p.armR.rotation.x;
  p.body.position.y=sit?-.4:Math.abs(Math.cos(ph))*.04*walk;
  p.body.rotation.x=slump;
  p.headG.rotation.y=headYaw;p.headG.rotation.x=headPitch;
  if(lie){p.g.rotation.x=-Math.PI/2;p.body.position.y=0;}
}
const faceTo=(p,dx,dz)=>Math.atan2(dx,dz); // model faces +z at rotation.y=0
function qpop(p,t,t0){const k=t-t0;p.q.scale.setScalar(k<0?0:.55*ease(k/.4)*(1+.18*Math.sin(k*12)*Math.max(0,1-k)))}

// ---- casts ----
const walkers=[];for(let i=0;i<34;i++){const st=rnd()<.6?pick(['xmm','xdd']):'adult';const p=person(st);p.x=R(-4.5,5.5);if(p.x>-1.5&&p.x<1.5)p.x+=3;p.z0=R(-40,16);p.dir=rnd()<.6?-1:1;p.v=R(.9,1.4);walkers.push(p)}
function updWalkers(t,count=99){walkers.forEach((p,i)=>{if(i>=count)return;p.g.visible=true;let z=p.z0+p.dir*p.v*t;z=((z+42)%60+60)%60-42;p.g.position.set(p.x,0,z);p.g.rotation.y=p.dir<0?Math.PI:0;pose(p,{walk:1,t})})}
const qY=[];for(let i=0;i<20;i++){const p=person(rnd()<.85?pick(['xmm','xdd']):'adult');p.x=-6.9+R(-.18,.18);p.z=-1.2+i*.78;p.phone=rnd()<.55;qY.push(p)}
const qM=[];for(let i=0;i<13;i++){const p=person(rnd()<.85?pick(['xmm','xdd']):'adult');p.x=R(-7.8,-4.8);p.z=R(-20.5,-13);p.ry=faceTo(p,-1,R(-.6,.6))+R(-.4,.4);qM.push(p)}
const sad=[];for(let i=0;i<20;i++){const p=person(i%2?'xmm':'xdd');p.cx=R(-5,4);p.cz=R(-22,4);p.r=R(.6,2.2);p.w=R(.12,.3)*(rnd()<.5?-1:1);sad.push(p)}
const sitters=[];for(let i=0;i<6;i++){const p=person(i%2?'xmm':'xdd');p.z=5-i*1.05;sitters.push(p)}
const lier=person('xdd');
const gM=[],gD=[];for(let i=0;i<5;i++){const a=person('xmm');a.x=-2.3+R(-.6,.5);a.z=-8+i*.9+R(-.3,.3);gM.push(a);const b=person('xdd');b.x=2.3+R(-.5,.6);b.z=-8+i*.9+R(-.3,.3);gD.push(b)}
const loner=person('xdd',{shirt:0x111111,pants:0x4a4436,scale:.95});
const bgSparse=[];for(let i=0;i<5;i++){const p=person(pick(['xmm','xdd','adult']));p.x=R(-4,4);p.z0=R(-40,-14);p.dir=rnd()<.5?-1:1;p.v=R(.6,1);bgSparse.push(p)}
const qC=[];for(let i=0;i<22;i++){const p=person(rnd()<.85?pick(['xmm','xdd']):'adult');p.tx=-7.3+R(-.15,.15);p.tz=-4.6+i*.8;p.sx=rnd()<.5?R(-3,4):R(5,8);p.sz=p.tz+R(10,22);p.arr=R(.6,4.6)+i*.03;qC.push(p)}
const builder=person('xdd',{shirt:0xffffff});

// ---- shots: s/e = timeline seconds; t inside cam/upd is shot-local ----
const camLook=(p,l)=>({p,l});
const SH=[
 {s:0,e:5.5,sky:'dusk',hud:null,caps:[],title:true,
  cam:t=>camLook(V(.4,1.75+Math.abs(Math.sin(t*3.75))*.03,14-t*1.3),V(-1.2,1.0,-6-t*1.3)),
  upd:t=>{updWalkers(t+3);qY.forEach(p=>{p.g.visible=true;p.g.position.set(p.x,0,p.z);p.g.rotation.y=Math.PI;pose(p,{t,armUp:p.phone?1.1:0,headPitch:p.phone?.4:0})});
    qM.forEach(p=>{p.g.visible=true;p.g.position.set(p.x,0,p.z);p.g.rotation.y=p.ry;p.cup.visible=true;pose(p,{t,armUp:1.0})});setShop(YOCHI,0,puffY,-1);setShop(MOLLY,0,puffM,-1)}},
 {s:5.5,e:12,sky:'dusk',hud:['TIME WITHOUT YO-CHI','Day 1','YO-CHI LEFT: 1'],hudSwap:[3.6,'YO-CHI LEFT: 0'],
  caps:[[0.3,3.2,'Day 1, 9pm: 80 xmms and xdds<br>queue for Yo-Chi.'],[4.1,6.5,'Then… <i>poof.</i>']],
  cam:t=>{const k=ease(t/6.5);return camLook(V(lerp(-.5,-2.2,k),lerp(2.0,1.85,k),lerp(7,3,k)),V(-9.5,2.4,-5.5))},
  upd:t=>{updWalkers(t+9,20);qY.forEach((p,i)=>{p.g.visible=true;p.g.position.set(p.x,0,p.z);p.g.rotation.y=Math.PI;const r=t>3.9+i*.05?ease((t-3.9-i*.05)/.5):0;pose(p,{t,armUp:p.phone?1.1*(1-r):0,headPitch:p.phone?.4*(1-r):0});});
    setShop(YOCHI,t<3.6?0:(t-3.6)/.35,puffY,(t-3.6)/1.6);setShop(MOLLY,0,puffM,-1)}},
 {s:12,e:18,sky:'dusk',hud:['TIME WITHOUT YO-CHI','Day 1','YO-CHI LEFT: 0'],
  caps:[[0.3,5.7,'Not one xmm or xdd<br>leaves the queue.']],
  cam:t=>{const k=t/6;return camLook(V(-4.2+k*.3,1.6,-5.5+k*.6),V(-6.8,1.45,5))},
  upd:t=>{setShop(YOCHI,2,puffY,2);setShop(MOLLY,0,puffM,-1);qY.forEach((p,i)=>{p.g.visible=true;p.g.position.set(p.x,0,p.z);p.g.rotation.y=Math.PI;
    const look=Math.sin(t*1.1+i)*.5*(i%3===0?1:0);pose(p,{t,headYaw:look,headPitch:-.05});qpop(p,t,.8+((i*7)%10)*.22)});}},
 {s:18,e:25.5,sky:'dusk',hud:['TIME WITHOUT YO-CHI','Day 2','MOLLY TEA LEFT: 1'],hudSwap:[3.4,'MOLLY TEA LEFT: 0'],
  caps:[[0.3,3.1,'Day 2: Molly Tea<br>is next.'],[4.2,7.2,'Even the cups<br>in their hands.']],
  cam:t=>{const k=ease(t/7.5);return camLook(V(lerp(-1.6,-2.8,k),1.8,lerp(-7.5,-9.5,k)),V(-8.5,1.9,-17.2))},
  upd:t=>{setShop(YOCHI,2,puffY,2);setShop(MOLLY,t<3.4?0:(t-3.4)/.35,puffM,(t-3.4)/1.6);
    qM.forEach((p,i)=>{p.g.visible=true;p.g.position.set(p.x,0,p.z);p.g.rotation.y=p.ry;const cupGone=t>4.4+i*.04;p.cup.visible=!cupGone;
      const stare=cupGone?ease((t-4.4-i*.04)/.4):0;pose(p,{t,armUp:1.0,headPitch:.55*stare,slump:.08*stare});qpop(p,t,4.9+(i%5)*.2)});
    updWalkers(t+20,10)}},
 {s:25.5,e:33,sky:'dusk',hud:['TIME WITHOUT YO-CHI','Day 3','XMMS UNACCOUNTED FOR: 2,400'],
  caps:[[0.3,3.6,'Day 3: the xmms and xdds<br>have nowhere to go.'],[4.0,7.2,'They just… wander.']],
  cam:t=>{const k=t/7.5;return camLook(V(2.5-k*1.2,5.2-k*.4,11-k*2),V(-1.5,0,-9-k*2))},
  upd:t=>{setShop(YOCHI,2,puffY,2);setShop(MOLLY,2,puffM,2);
    sad.forEach(p=>{p.g.visible=true;const a=t*p.w+p.phase;p.g.position.set(p.cx+Math.cos(a)*p.r,0,p.cz+Math.sin(a)*p.r);p.g.rotation.y=Math.atan2(-Math.sin(a)*Math.sign(p.w),Math.cos(a)*Math.sign(p.w));pose(p,{walk:.45,speed:4.2,t,headPitch:.45,slump:.15})});
    sitters.forEach(p=>{p.g.visible=true;p.g.position.set(6.4,.09,p.z);p.g.rotation.y=-Math.PI/2;pose(p,{t,sit:true,headPitch:.7,slump:.35})});
    lier.g.visible=true;lier.g.position.set(1.2,.15,-4);lier.g.rotation.y=.6;pose(lier,{t,lie:true});}},
 {s:33,e:40.5,sky:'dusk',hud:['TIME WITHOUT YO-CHI','Week 1','AWKWARDNESS: 100%'],
  caps:[[0.3,3.6,'Week 1: xmms and xdds,<br>once inseparable…'],[4.1,7.2,'…are now estranged.']],
  cam:t=>{const k=t/7.5;return camLook(V(0,1.8,2.8-k*1.0),V(0,1.05,-7))},
  upd:t=>{setShop(YOCHI,2,puffY,2);setShop(MOLLY,2,puffM,2);const turn=ease((t-4.1)/.8);
    gM.forEach(p=>{p.g.visible=true;p.g.position.set(p.x,0,p.z);p.g.rotation.y=lerp(Math.PI/2,-Math.PI/2+.3,turn);pose(p,{t,headPitch:.1,slump:.05})});
    gD.forEach(p=>{p.g.visible=true;p.g.position.set(p.x,0,p.z);p.g.rotation.y=lerp(-Math.PI/2,Math.PI/2-.3,turn);pose(p,{t,headPitch:.1})});
    tumble.visible=true;const tx=lerp(-6,6,clamp((t-1.2)/5.5,0,1));tumble.position.set(tx,.45+Math.abs(Math.sin(t*5))*.25,-4.5);tumble.rotation.z=-tx*1.5;}},
 {s:40.5,e:47,sky:'dusk',hud:['TIME WITHOUT YO-CHI','Week 2','HOPE: 3%'],
  caps:[[0.3,3.4,'Week 2: one xdd still carries<br>his empty Yo-Chi bowl.'],[3.9,6.3,'Out of respect.']],
  cam:t=>{const k=ease(t/6.5);return camLook(V(lerp(1.6,2.8,k),lerp(1.7,1.45,k),lerp(1.2,-.4,k)),V(6.6,0.9,-2.6))},
  upd:t=>{setShop(YOCHI,2,puffY,2);setShop(MOLLY,2,puffM,2);loner.g.visible=true;loner.g.position.set(6.4,.11,-2.6);loner.g.rotation.y=-Math.PI/2;loner.bowl.visible=true;
    pose(loner,{t,sit:true,armUp:.9,headPitch:.5+Math.sin(t*.8)*.05,slump:.12});
    bgSparse.forEach(p=>{p.g.visible=true;let z=p.z0+p.dir*p.v*t;p.g.position.set(p.x,0,z);p.g.rotation.y=p.dir<0?Math.PI:0;pose(p,{walk:.6,speed:5,t,headPitch:.3})})}},
 {s:47,e:54,sky:'dusk',hud:['TIME WITHOUT YO-CHI','Month 1','HOPE: 81%'],
  caps:[[0.3,3.3,'Month 1: an xdd opens<br>a Yo-Chi made of cardboard.'],[3.8,6.7,'The queue is 200 people.<br>Again.']],
  cam:t=>{const k=ease(t/7);return camLook(V(lerp(-.6,.4,k),lerp(2.1,2.5,k),lerp(-.5,2.5,k)),V(-7.2,1.2,-3.2))},
  upd:t=>{setShop(YOCHI,2,puffY,2);setShop(MOLLY,2,puffM,2);stall.visible=true;
    builder.g.visible=true;builder.g.position.set(-9.4,0,-5.6);builder.g.rotation.y=Math.PI/2;builder.bowl.visible=true;pose(builder,{t,armUp:1.2+Math.sin(t*3)*.2});
    qC.forEach(p=>{p.g.visible=true;const k=clamp((t-p.arr+2.2)/2.2,0,1);const x=lerp(p.sx,p.tx,ease(k)),z=lerp(p.sz,p.tz,ease(k));p.g.position.set(x,0,z);
      p.g.rotation.y=k<1?Math.atan2(p.tx-p.sx,p.tz-p.sz):Math.PI;pose(p,{walk:k<1?1:0,speed:11,t})});}},
 {s:54,e:60,sky:'night',hud:['TIME WITHOUT YO-CHI','Day 47','HOPE: ???'],
  caps:[[0.4,3.4,'Then one night…<br>the lights come back on.']],
  cam:t=>{const k=ease(t/5);return camLook(V(1.2,2.3,9.5-k*1.5),V(-6.5,2.0,-5))},
  upd:t=>{const k=clamp((t-1.2)/.8,0,1);const sx=ease(k);YOCHI.g.visible=sx>0;YOCHI.g.scale.set(sx,1,sx);YOCHI.ghost.visible=sx<1;puffY(-1);setShop(MOLLY,2,puffM,2);
    glowL.intensity=k>0?60*(1+.5*Math.sin(t*20)*(1-k)):0;stall.visible=true;
    qC.concat(sad.slice(0,12)).forEach((p,i)=>{p.g.visible=true;const base=i<qC.length?V(p.tx,0,p.tz):V(p.cx,0,p.cz+6);const go=clamp((t-2.0-i*.02)/3.5,0,1);
      const tx=-7.5,tz=-6+(i%6)*.4;const x=lerp(base.x,tx,go),z=lerp(base.z,tz,go);p.g.position.set(x,0,z);p.g.rotation.y=go>0?Math.atan2(tx-base.x,tz-base.z):Math.PI;
      pose(p,{walk:go>0&&go<1?1:0,speed:14,t,cheer:t>1.8&&t<2.4?ease((t-1.8)/.3):0})});}},
];
const capEl=document.getElementById('cap'),titleEl=document.getElementById('title'),hudEl=document.getElementById('hud'),blackEl=document.getElementById('black'),endEl=document.getElementById('end'),hudbg=document.getElementById('hudbg'),capbg=document.getElementById('capbg');
const hk=document.getElementById('hudk'),hv=document.getElementById('hudv'),hm=document.getElementById('hudm');
function fade(t,a,b,f=.3){return clamp(Math.min((t-a)/f,(b-t)/f),0,1)}
window.renderAt=function(T){
  const sh=SH.find(s=>T>=s.s&&T<s.e)||SH[SH.length-1];const t=T-sh.s;
  all.forEach(p=>{p.g.visible=false;p.cup.visible=false;p.bowl.visible=false;p.q.scale.set(0,0,0)});
  stall.visible=false;tumble.visible=false;glowL.intensity=0;
  const night=sh.sky==='night';scene.background=night?SKY_NIGHT:SKY_DUSK;hemi.intensity=night?.45:1.1;sun.intensity=night?.25:1.6;scene.fog.color.set(night?0x111a35:0x8a86b4);
  stripMat.color.set(night?0xfff0c0:0xfff6e0);
  drawLED(T);sh.upd(t);
  const c=sh.cam(t);camera.position.copy(c.p);camera.fov=c.fov||58;camera.updateProjectionMatrix();camera.lookAt(c.l);
  post.bloom.strength=sh.bloom??.38;renderer.toneMappingExposure=sh.exposure??1.05;post.bokeh.enabled=!!sh.dof;if(sh.dof)post.bokeh.uniforms.focus.value=c.focus||camera.position.distanceTo(c.l);
  titleEl.style.opacity=sh.title?fade(t,-1,5.2,.4):0;
  if(sh.hud){hudEl.style.opacity=1;hk.textContent=sh.hud[0];hv.textContent=sh.hud[1];hm.textContent=sh.hudSwap&&t>=sh.hudSwap[0]?sh.hudSwap[1]:sh.hud[2];}else hudEl.style.opacity=0;
  hudbg.style.opacity=hudEl.style.opacity;
  let html='',op=0;for(const [a,b,s] of sh.caps){const f=fade(t,a,b);if(f>0){html=s;op=f}}capEl.innerHTML=html;capEl.style.opacity=op;capbg.style.opacity=op;
  const last=sh===SH[SH.length-1];blackEl.style.opacity=last?clamp((t-3.7)/.5,0,1):0;endEl.style.opacity=last?fade(t,4.1,6.2,.3):0;
  post.composer.render();
};
window.ready=true;
</script></body></html>
```

## Gotchas
- Fonts must be loaded (`document.fonts.load`) before any canvas `label()` is drawn, or signs render in a fallback font. The template does this with a top-level await.
- Never use requestAnimationFrame or wall-clock time. Every frame is a pure function of `renderAt(T)`, so shots can render in parallel and the result is deterministic.
- Shell network access is limited to package registries. If the user wants a trending sound, ask them to attach the audio and mux it instead of music.wav.
- Write each shot to `*.tmp.mp4` and rename it only when ffmpeg exits 0, so a stopped render never leaves a half file.
- Never write a dollar sign followed by a digit anywhere in this skill. When the skill is invoked with text after the command, that text is substituted into those spots, which broke runall.sh twice; the template uses `${BASH_SOURCE[0]}`, environment variables and `"$@"` instead.
- Shot state must not leak between shots: each shot sets its own camera fov, set visibility, mood (lights, emissives, sky, fog), colours, hair sizes and prop visibility, because parallel jobs render shots out of order.
- `new THREE.Box3().setFromObject(obj, true)` gives tight bounds. Without `true` it uses rotated bounding boxes and overshoots, which once put a "standard height" line above everyone.
- On SwiftShader `gl.finish()` returns before the frame is drawn, so time frames by the screenshot (render.js logs averages every 30 frames).
- LatheGeometry revolves around y; `geometry.rotateZ(Math.PI / 2)` lays the profile along −x, with the nose at the origin.
- 3D pop-ups like the '?' sprites use `depthTest: false` and a high `renderOrder` so scenery never hides them.