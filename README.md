# 🍂 Leaf Classifier — MLR exercise 08

**Open the app:** <https://257010.github.io/mlr-ex08/>

Live leaf classification running **entirely in the browser** — point your **phone's rear
camera** at a leaf and see the probability of every class update in real time. Works in
Safari on iPhone (iOS 16.4+), Android Chrome and desktop browsers. No server, no data
upload: inference happens locally via [ONNX Runtime Web](https://onnxruntime.ai/web)
(WASM).

## How to use

1. Open <https://257010.github.io/mlr-ex08/> on your phone (or scan the QR code at the
   bottom of the page).
2. Tap **▶ Start rear camera** and allow camera access.
3. Hold a leaf so it fills the centre of the frame — the small inset shows the exact
   96×96 image the model receives.
4. If the camera cannot be used, tap **📷 Upload photo** — the same pipeline runs on the
   still image.

## Model

`leaf_model.onnx` — CNN trained on four leaf classes (`model_info.json`):

| class | apple · cherry · chestnut · maple |
|---|---|

Preprocessing (done client-side, exactly as specified in `model_info.json`):

1. crop the **central square** of the camera frame,
2. resize it to **96 × 96** (`input_size`),
3. scale RGB pixels to **0–1**,
4. standardise each channel: `(x − mean) / std` with
   `mean = [0.485, 0.456, 0.406]`, `std = [0.229, 0.224, 0.225]`,
5. feed a float32 tensor of shape **1 × 3 × 96 × 96** (NCHW).

The model outputs `logits`; the page applies softmax and displays the probability of
every class.

**Self-check:** uploading `check_image.jpg` reproduces the probabilities in
`check_expected.json` (apple ≈ 0.9989591, cherry ≈ 0.0010149, chestnut ≈ 2.156e-5,
maple ≈ 4.572e-6).

## Files

| file | purpose |
|---|---|
| `index.html` | the whole app — plain JavaScript, ONNX Runtime Web from CDN |
| `leaf_model.onnx` | the trained classifier |
| `model_info.json` | classes, input size, channel mean/std |
| `check_image.jpg` / `check_expected.json` | verification image + expected probabilities |

## Notes

* The QR code at the bottom of the page is generated **locally in the browser** with the
  MIT-licensed [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator)
  library (embedded in `index.html`) — no external QR/image service is used.
* Camera access requires HTTPS (GitHub Pages provides it).
* Uses `onnxruntime-web` 1.31.1 from jsDelivr, WASM backend, single thread
  (iOS/Safari-friendly).

*MLR (Machine Learning in Practice) exercise 08 — ONNX deployment.*
