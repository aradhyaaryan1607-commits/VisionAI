# VisionAI

A voice-controlled web app that helps a blind person move safely and identify things. A camera mounted on spectacles (ESP32-CAM, phone or webcam) streams video to the page. The app speaks warnings and answers in **Hindi, Hinglish or English**.

Only two things are touched: **camera selection** and **language selection**. Everything else is voice.

---

## Features

| Feature | What it does |
|---|---|
| Obstacle alert | Speaks only when something in the walking path is **1 m or closer** |
| Wall / large surface | Warns when a large plain surface fills the view ahead |
| Vehicle warning | Warns when a car, truck, bus, bike or cycle is coming towards the person fast |
| Find an object | "pani ka bottel kha hai" finds the keyword and says where it is |
| Currency | "kitne ka note hai ye" matches the note or coin against your photos |
| Products | "ye kya hai" matches the item against your product photos |
| Sign boards | "board padho" matches sign photos |
| Live boxes | Detected obstacles are boxed on screen with name and distance |

---

## Quick start

1. Put `index.html` in the root of this repository.
2. Create the `objects/` folder with photos (see below).
3. Go to **Settings → Pages**, choose the main branch and root folder, then save.
4. Open the Pages link in **Chrome on Android** (voice recognition works best there).
5. Choose the camera and language, then tap **Start camera**. Allow camera and microphone.
6. Wait for "Ready", then say **"start"** (or "shuru").

Until the person says "start", the app does nothing. After that it stays silent except for dangers within 1 m or a fast-approaching vehicle, until he asks for something.

---

## Photo folder layout

```
objects/
  indoor/
    chair/        chair_1.jpg  chair_2.jpg  chair_3.jpg
    water_bottle/ water_bottle_1.jpg  water_bottle_2.jpg
  products/
    parle_g/      parle_g_1.jpg  parle_g_2.jpg
  currency/
    note_10/      note_10_1.jpg  note_10_2.jpg
    note_500/     note_500_1.jpg
    coin_5/       coin_5_1.jpg
  signs/
    exit/         exit_1.jpg
    no_entry/     no_entry_1.jpg
  obstacles/
    cupboard/     cupboard_1.jpg
    door/         door_1.jpg
    pole/         pole_1.jpg
  aliases.json    (optional)
```

### Naming rules

- Folder pattern: `objects/<category>/<item_name>/<item_name>_<number>.jpg`
- Categories must be exactly: `indoor`, `products`, `currency`, `signs`, `obstacles`.
- The item folder name is what the app speaks. Use lowercase and underscores (`water_bottle` is spoken as "water bottle").
- Currency items must be named `note_<value>` or `coin_<value>`, for example `note_500` or `coin_5`. The app then says "Ye 500 rupaye ka note hai".
- Allowed image types: `.jpg`, `.jpeg`, `.png`, `.webp`.
- Use **5 to 10 photos per item**: front, back, angled, bright light, dim light, on a plain background and in hand. More variety gives better matching.
- Keep photos small (about 640 px wide) so loading is fast.

### Optional `objects/aliases.json`

Adds extra words the person might say, and an optional real height in **cm** used for distance:

```json
{
  "water_bottle": { "alias": ["pani", "bottel", "botal"], "h": 25 },
  "parle_g": { "alias": ["parle", "biscuit"], "h": 15 },
  "cupboard": ["almirah", "alamari"]
}
```

If the folder listing does not load, add `objects/manifest.json`, a list of image paths:

```json
["objects/indoor/chair/chair_1.jpg", "objects/currency/note_500/note_500_1.jpg"]
```

---

## Voice commands

The app listens for **keywords**, not full sentences.

| Say (examples) | Keyword | Result |
|---|---|---|
| start, shuru, chalu | start | Begins obstacle watching |
| stop, ruko, band | stop | Pauses everything |
| kitne ka note hai, sikka kitne ka, how much rs is this | note, sikka, coin, rs, kitne ka | Currency check |
| board padho, what does the sign say | board, sign, likha | Sign check |
| ye kya hai, product batao, saman | product, saman, kya hai | Product check |
| pani ka bottel kha hai, where is the chair | an object name or alias | Finds it and says left, front or right, plus distance |

Hold notes, products and signs about 20 to 40 cm in front of the camera, centred.

Built-in objects the camera already knows (no photos needed): person, chair, bottle, cup, sofa, bed, table, TV, laptop, phone, book, car, truck, bus, bike, cycle, dog, cat, plant, fridge and more. Your photos can add anything else.

---

## ESP32-CAM setup

1. Flash the standard **CameraWebServer** example.
2. Note its IP address, for example `192.168.4.1`.
3. In the app choose **ESP32-CAM** and enter `http://<ip>:81/stream`.
4. GitHub Pages is `https` but the ESP32 stream is `http`, so Chrome blocks it. Fix it by either:
   - Chrome → site settings for the page → **Insecure content → Allow**, or
   - opening `index.html` locally on a device on the same Wi-Fi.
5. The stream must send CORS headers (the standard sketch does).

---

## Calibration (for the programmer)

Open the page with `?dev=1` for sliders, or edit the top of the script in `index.html`:

```js
const C = { F: .95, THR: .62, WALL: 5, REPO: '' };
```

| Setting | Meaning |
|---|---|
| `F` | Camera focal length (times frame height). Raise or lower until the spoken distance matches a tape measure |
| `THR` | Photo match strictness, from 0 to 1. Higher means fewer wrong matches but more "nahi mila" |
| `WALL` | Wall sensitivity. Higher flags more plain surfaces |
| `REPO` | Set to `owner/repo` if photo auto-listing fails |

The `H` table in the script holds real heights (in metres) of common objects. Edit it to improve distance accuracy.

To calibrate distance: place a known object (for example a 25 cm bottle) at exactly 1 m, then adjust `F` until it is announced at about 1.0 meter.

---

## Known limits

- **Distance is an estimate** from object size, not a true measurement. Items not in the `H` table or `aliases.json` get no distance.
- **Walls**: one camera cannot measure wall distance. The app only detects large plain surfaces, so it can fire early or miss textured walls. For reliable 1 m wall warnings add an ultrasonic sensor (HC-SR04) to the ESP32.
- **Vehicles**: the warning uses how fast the vehicle grows in the frame. Walking towards a parked vehicle can also trigger it.
- Photo matching can be wrong in poor light. Add more photos if it confuses items.
- This is an assistive aid, **not a replacement for a cane or guide**. Test thoroughly before real-world use.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| No voice response | Use Chrome on Android, allow the microphone, check device volume |
| "Camera error" | Allow camera permission, or check the ESP32 URL and Wi-Fi |
| ESP32 video is blank | Allow insecure content (see ESP32 setup) |
| Photos not loading | Check folder names and categories, add `manifest.json`, or set `REPO` |
| Wrong items matched | Add more photos or raise `THR` |
| Wrong distances | Recalibrate `F` |

---

## Tech

Runs fully in the browser with TensorFlow.js, COCO-SSD (object detection), MobileNet (photo matching), the Web Speech API (voice in and out). No server and no data leaves the device, except the page asking GitHub for the photo list.
