# RantPilot

RantPilot turns one long YouTube video into a short course page. You paste a video link and a transcript, the app splits the video into timed clips, writes three headline choices, and can add a button that appears after a delay. Save to Cloud gives you a public link.

The screens and words follow the Stitchable walkthrough (`stitchable.99dfy.com`). Reference pictures are in `docs/reference/`.

## Run it on Windows/Laragon

Do these steps once, in order. Open **Laragon**, click **Start All**, then open the **Terminal** in the project folder `C:\laragon\www\rantpilot`.

1. **Install PHP packages.** Type `composer install` and press Enter. Wait until it finishes.
2. **Install the page styles.** Type `npm install` and press Enter.
3. **Make the settings file.** Type `copy .env.example .env` and press Enter.
4. **Create the secret app key.** Type `php artisan key:generate` and press Enter.
5. **Create the database.** In Laragon click **Database**, then create a database named `rantpilot`. The user is `root` and the password is empty. That matches `.env`.
6. **Create the tables and the demo course.** Type `php artisan migrate --seed` and press Enter.
7. **Start the website.** Type `php artisan serve --port=8001` and leave that window open. Open http://127.0.0.1:8001 in the browser. Port 8000 is already used by YOUWATCH, so we use 8001.
8. **Start the styles (second window).** Open a second terminal in the same folder and type `npm run dev`. Leave it open while you work.

Log in with:

- Email: `demo@rantpilot.test`
- Password: `password`

You will land on **My Stitchables**. Open **Food truck** (the demo). The public page is http://127.0.0.1:8001/s/food-truck

### The background worker (third window)

Clip generation runs in the background so the page does not freeze. Open a third terminal and type:

```
php artisan queue:work
```

Leave it open. If you skip this, **Generate Timestamps** will sit on 0% and the page will tell you to start the worker.

One shortcut does the site, the worker, and the styles together (still on port 8001):

```
composer run dev
```

### Tests

```
php artisan test
```

Tests use SQLite in memory. They do not touch your MySQL database.

### Optional: Groq

To use Groq for the clips and headlines, put your key in `.env`:

```
GROQ_API_KEY=your-key-here
```

Leave it empty and the app still builds clips on its own, with no internet call. Never commit a real key.

### ffmpeg

Avatar Studio's photo mode needs ffmpeg, including ffprobe, on the PATH. In Laragon you can use a Windows build from gyan.dev and add its `bin` folder to PATH. Check with `ffmpeg -version`.

### Chatterbox (your own voice, optional)

Chatterbox is a local voice clone from Resemble AI. The licence is MIT. This PC is an Intel i5-8265U with no GPU, so install the CPU build of torch. The first run downloads a model of a few GB. Free disk space first if you are near 5 GB free. Kokoro is the smaller fallback. It does not clone your voice.

Open a terminal in `C:\laragon\www\rantpilot` and run these commands in order:

```
cd C:\laragon\www\rantpilot
py -3 -m venv tools\voiceclone\.venv
tools\voiceclone\.venv\Scripts\activate
python -m pip install --upgrade pip
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install chatterbox-tts
php artisan avatar:setup-check
```

Optional Kokoro fallback, from the same activated venv:

```
pip install kokoro soundfile
```

`php artisan avatar:setup-check` prints PASS or FAIL for ffmpeg, Python, Chatterbox, Kokoro, free disk, and lip-sync. The same list is on **Avatar Studio → Setup check**. On this PC the Chatterbox import took about 30 seconds, so the check waits 120 seconds (`AVATAR_ENGINE_CHECK_TIMEOUT` in `.env`). A successful check is remembered for a day, or until you change the voice venv, so the other pages do not wait again.

Auto uses Chatterbox when that check passes. It will not switch to Kokoro. If you want Kokoro, choose it on the form. Kokoro is not your voice.

Expressiveness and voice match default to 0.4 and 0.35 (`AVATAR_EXAGGERATION` and `AVATAR_CFG`). Avatar Studio can override them per render. Voiceover uses the same defaults on its sliders. Both send the numbers to Chatterbox.

To run Chatterbox on another machine, set `VOICE_ENGINE_URL` and `VOICE_ENGINE_TOKEN` in `.env`. The server script is `tools/voice-server/server.py`. Leave the URL blank to stay on this PC. Kokoro stays on this PC either way. Setup check shows the server host and does not show the token.

### Moving avatar

The free default is **Moving avatar (free, Kaggle)** on Avatar Studio. It zips your photo and a voiceover for the InfiniteTalk notebook. Put `InfiniteTalk_Kaggle.ipynb` in `tools/infinitetalk/` if you want that file inside the zip. Upload the finished MP4 back on the same page. It lands in your library and Editor Studio can import it.

Fast GPU is optional. It is off, and the page says **Fast GPU: not configured**, until `RUNPOD_API_KEY` and `RUNPOD_ENDPOINT_ID` are set. That path is paid per video, billed by RunPod. The worker notes are in `tools/runpod-infinitetalk/README.md`.

Use only your own face and voice. On YouTube, apply the altered or synthetic content label.

On this CPU, Chatterbox is about 4 to 6 times the length of the speech. One minute of talking can take 4 to 6 minutes to voice, before ffmpeg draws the picture. Keep `php artisan queue:work` open while a render or a voice zip is queued.

Photo mode does not move the mouth. It is not lip-sync. You can upload your own talking-head clips (a phone recording, or a lip-sync you made elsewhere) and assign them to the hook, a scene, the CTA, or the outro. **Export scene audio** zips a wav for each part plus your photo, so you can lip-sync those in another tool and upload the videos back. RantPilot does not remove watermarks or logos.

Paid lip-sync stays off unless you set `REPLICATE_API_TOKEN` in `.env`. You do not need it.

### Voiceover

Voiceover speaks a script in your cloned voice. It uses the Chatterbox install from Avatar Studio. There is no second setup.

1. Sign in.
2. Open **Voiceover**. If the page says Chatterbox is not installed, open **Voice setup** and run the Windows commands above.
3. In Avatar Studio, save a profile and tick **This is my own face and voice**. That profile is the default voice here. Kokoro appears only when it is installed, and it does not clone you.
4. Paste the script. The line under the box shows characters, words, and about how long it will take (2.5 words a second).
5. Click **Clean script for speech** if the script has prices, numbers, or web addresses. A blank line is a longer pause. Type `[pause 1s]` for a one second pause.
6. Leave Expressiveness at 0.40 and Voice match at 0.35, or click **Reset to recommended**. Those are the approved settings, and they are what Chatterbox receives.
7. Click **Generate voiceover**. Leave a terminal open on `php artisan queue:work --tries=1 --timeout=0`. The chunk list shows how far it got. If it stops, click **Resume**. Finished chunks are kept.
8. Play it in the browser, or download the WAV or the MP3. **Word timings** is a JSON file for captions.
9. **Send to Avatar Studio** uses this audio under your photo. **Send to Editor Studio** puts it in the media bin and on the Voiceover track. **Use in commentary slot** copies the spoken words into a Reaction Blueprint slot.

This audio carries Chatterbox's inaudible watermark. RantPilot does not remove it. On this PC, Chatterbox on CPU takes about 4 to 6 times as long as the speech.

### Editor Studio

Editor Studio is the reaction timeline. It uses the same ffmpeg binary as Avatar Studio. There is no extra install.

1. Sign in.
2. Open **Editor Studio** and create a project. Pick 16:9 or 9:16.
3. The screen is the desktop layout: **Media**, **Text**, **Audio**, and **Layouts** on the left, the player in the centre, properties on the right, and the timeline along the bottom.
4. Upload a video, image, or audio file. Place it on Source or Reactor. You can also import an Avatar Studio render, one of your own avatar clips, or a Reaction Blueprint plan.
5. **Import reaction plan** drops the hook, then each source card alternating with a reactor slot, then the CTA and the outro. A source card longer than 15 seconds shows a 15s+ badge.
6. Use the layout buttons for full-screen source, full-screen reactor, picture-in-picture, side by side, or top/bottom. Drag to move, drag the edges to trim, and use Split, Duplicate, Delete, Ripple delete, Undo, and Redo. Magnet is the snap toggle. The zoom slider sits on the timeline.
7. Shortcuts: Space plays and pauses. S or Ctrl+B splits. Delete removes. Ctrl+Z undoes. Ctrl+Shift+Z redoes. Ctrl+C and Ctrl+V copy and paste. The arrow keys step one frame.
8. On Text, add a metric callout, a glowing question mark, the "Unpaid Overtime Energy" label, a lower third, or a subscribe reminder. Paste a transcript and click **Orange highlight captions**.
9. On Audio, add a CC0 sound (whoosh, pop, record scratch, snore, ding). Music can duck under speech. A voice clip can be normalised.
10. Leave a terminal open on `php artisan queue:work`. That worker also builds the low-res proxy.
11. Click **Export 1080p**. When the row says Ready, click **Download**.

The preview plays a 640px proxy. The download is 1080p, built in short pieces: H.264 Main, yuv420p, AAC, fast start. RantPilot does not remove watermarks or logos. A Reaction Blueprint source card is a title, because the YouTube video is not downloaded. Chroma key is a green screen, not an AI cutout.

#### How to overlay, crop, freeze and react, and change volume

1. Upload the video you are reacting to and click **Main**. Upload your face cam, or a picture, and click it in the bin so it is highlighted.
2. Move the playhead to the moment you want. Click **Overlay**. The picture lands on Overlay 1, above the main track. Overlay 2 and Overlay 3 sit further in front. Drag a bin item onto a track if you would rather drop it there.
3. The player draws that overlay on top of the main video: pictures, videos, snapshots, and freeze frames. Drag it to move it. Drag the white corner to resize it, and the orange **R** handle to rotate it. The properties column has X, Y, Scale, Rotation, and Opacity. **Reset transform** puts it back.
4. Click **Crop**, then drag the white box. Or pick a preset: Free, 16:9, 9:16, 1:1, 4:3, Circle, or Rounded. **Reset crop** clears it. The original file is not cut; the crop is saved with the project and applied on export.
5. To react on a still: park the playhead on the main clip and click **Freeze frame**. The default hold is 3 seconds (change **Freeze length**). The video continues after the hold. **Snapshot** saves that frame as a PNG in the bin, so you can drop it on the main track or on an overlay and talk over it. **Freeze + react** does the hold, quiets the source for those seconds, and adds a circle face-cam slot plus a "React here" caption.
6. Every clip with sound has a volume slider from 0% to 200%, a mute box, and fade in / fade out. The percent sits on the clip. Each track header has its own mute button and volume slider. **Detach audio** moves a video's sound onto the audio track. Music still ducks under speech.

Right-click a clip for split, delete, duplicate, freeze, detach audio, or switch to overlay. **Bring forward** and **Send back** change the overlay layer. **Switch to overlay** / **Switch to main track** moves the selected clip. Ripple delete only closes the gap on the main track. Shortcuts are listed under **Shortcuts** on the timeline: Ctrl+B split, Delete, Ctrl+D duplicate, F freeze, M mute.

## What you can do

- **New** opens a blank course.
- **Bulk Import** pastes a YouTube URL and a tactiq.io transcript, then **Generate Timestamps**.
- **Import Clips** reads the Time Ranges box (`00:01:30 → 00:02:45`, or a comma, or a hyphen).
- **+ Single Clip** adds one clip by hand.
- Pick one of the three **Marketing Headlines**.
- Style the timed button and click **+ Add Timed Button**. It shows on the public page after the delay.
- **Save to Cloud** publishes `/s/{slug}`.
- **Export Json** / **Import Json** download and restore a course.
- **My Stitchables** lists your courses. You can open, copy the public link, or delete.
- **Reaction Blueprint** (inside the editor) cuts a transcript into short clips, then gives you commentary slots, a checklist, and a script download.
- **Copy NotebookLM prompt** builds a prompt from those cuts. Paste the numbered answer back into **Paste NotebookLM answer**.
- **Avatar Studio** turns a script into a video from your photo and voice, or from clips you upload.
- **Editor Studio** is the reaction timeline: a main track, a face cam, three overlay tracks, text, and audio, plus crop, freeze, snapshot, per-clip volume, a low-res proxy, and a queued 1080p export.
- **Transcribe** and **Run RANT** are placeholders. They are not built yet.
- **Dark** switches the editor theme.

## Docs

- `CLAUDE.md` is the original screen spec.
- `docs/ASSUMPTIONS.md` lists every choice that the spec left open.
- `docs/ARCHITECTURE.md`, `docs/DATABASE_SCHEMA.md`, `docs/ENVIRONMENT.md`, `docs/SECURITY.md`.
- `docs/KNOWN_ISSUES.md` is what is still unfinished.
