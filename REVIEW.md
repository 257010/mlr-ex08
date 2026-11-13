# REVIEW – Task 2: Leaf classifier web application

## 1. Testing the application on a mobile phone

I opened the application on a mobile phone via GitHub Pages:

https://257010.github.io/mlr-ex08/

The page loaded and I allowed it access to the camera. I tested the classification using the
live image from the camera. The application displayed the predicted class and the probabilities
of the individual classes. The result updated according to the leaf being captured.

I documented the test with the screenshot `phone_test.jpg`, which I added to the GitHub
repository and to the folder with the Task 2 materials.

## 2. Test using the reference image

In the application I used the provided reference image `check_image.jpg`. The model classified
it as **apple** with a probability of approximately **99.9 %**.

Comparison of the displayed values with the reference values in `check_expected.json`:

| Class | Reference probability | Displayed value after rounding |
|---|---:|---:|
| apple | 0.998959 | 0.999 (99.9 %) |
| cherry | 0.001015 | 0.001 (0.1 %) |
| chestnut | 0.000022 | 0.000 (0.0 %) |
| maple | 0.000005 | 0.000 (0.0 %) |

The result corresponds to the reference output. The small differences are caused by rounding of
the displayed probabilities.

## 3. Checking the image preprocessing

In the `index.html` file I checked the processing of the input image before it is passed to the
model. The image is first cropped to a square from the centre, then it is resized to
**96 × 96 pixels**. The pixel values are converted to the range 0 to 1 by dividing by 255 and
then normalised using the means and standard deviations of the individual colour channels.

The values used are:

- mean: `[0.485, 0.456, 0.406]`
- std: `[0.229, 0.224, 0.225]`

The input is created in **RGB** channel order and in the format that the model expects. These
settings correspond to the data in `model_info.json` and to the preprocessing used when the
data were prepared in the notebook. The input in the application is therefore processed in the
same way as during the training of the model.

The exact places in `index.html` are these two functions, next to each other near the end of
the file:

- **`preprocessToImageData()` (lines 2524 – 2541)** — cropping and resizing. It takes the
  central square of the frame (`side = Math.min(srcW, srcH)`, centred by the offsets `sx` and
  `sy`) and reduces it to `INFO.input_size` × `INFO.input_size`, i.e. 96 × 96. The reduction
  happens in steps of at most one half at a time, so that a large camera frame is not aliased
  by a single jump, and the last step draws the 96 × 96 result into the small preview box —
  what you see in that box is exactly the image the model receives.
- **`imageDataToTensor()` (lines 2543 – 2554)** — scaling, standardising and the layout of the
  tensor. It reads the pixel data, divides each channel by 255, then computes
  `(x − INFO.mean[c]) / INFO.std[c]` for red, green and blue, writes the three channels one
  after another into a `Float32Array` — planar **CHW** order: `t[i]` is red, `t[n + i]` green,
  `t[2 * n + i]` blue — and returns `ort.Tensor("float32", t, [1, 3, S, S])`, the shape
  1 × 3 × 96 × 96 that the model expects.

Both functions take their numbers from `INFO`, which is the fetched `model_info.json` (loaded at
start-up, with an embedded fallback holding the same values), so the values used in the code
cannot drift away from the metadata of the model.

## 4. Privacy and image transfer

During the testing I checked how the application works. The classification itself runs directly
in the browser using ONNX Runtime Web and the `leaf_model.onnx` file. Neither the image from
the camera nor an uploaded image is sent to a classification server of our own. The application
therefore does not need to send the user's photographs outside the device in order to perform a
prediction.

What is loaded are the application files from GitHub Pages and the ONNX Runtime Web library
from an external CDN. These resources serve to load the web page and the model and to perform
the inference in the browser. Access to the camera is needed only for the live capture and the
user has to allow it in the browser.

## Conclusion

I tested the application on a mobile phone with both the live camera and the reference image.
The live camera worked and the reference image `check_image.jpg` was classified correctly as
`apple` with a probability of approximately 99.9 %. The check of the code confirmed that the
preprocessing of the input corresponds to the parameters of the model. The classification runs
locally in the browser and the images are not sent to a classification server.
