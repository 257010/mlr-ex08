# REPORT – Task 4: Experiments and improvements

## 1. Objective

The goal of Task 4 was to explore the options for improving the leaf classification and to
compare their effect on the validation set and on an additional set of images transformed so
that they roughly simulate the conditions of taking a photograph with a mobile phone. As the
starting point I used my own convolutional neural network, LeafNet, from Task 3.

When comparing the variants I mainly tracked the validation accuracy and the accuracy on the
camera-like set. The latter was created by applying several image transformations to the
original validation images. It is therefore not a completely independent set of real
photographs from a phone, but a supplementary test of robustness to changes of the image.

## 2. Modifications tried

In turn I tried data augmentation, normalisation using Batch Normalization, colour
adjustment, dropout, the AdamW optimiser, the pretrained MobileNetV3 Small network, fine-tuning
of its weights, planning of the learning rate with a cosine scheduler, Global Average Pooling
and label smoothing.

In the experiments with LeafNet I always changed one specific property of the model or of the
training, so that its effect could be compared with the baseline solution. For MobileNetV3
Small I compared the frozen pretrained network with variants in which part or all of the
weights were trained further.

## 3. Results

In the table I report the mean accuracy and the standard deviation over repeated runs, where
these were available. In the MobileNetV3 Small experiments some variants were repeated twice;
the final experiment 9 was performed as a single run. The values are given as a fraction, not
as percentages.

| Experiment | Modification | Validation accuracy | Camera-like accuracy |
|---|---|---:|---:|
| 0 | LeafNet – baseline model | 0.672 ± 0.041 | 0.406 ± 0.043 |
| 1 | LeafNet + geometric augmentation | 0.694 ± 0.043 | 0.648 ± 0.045 |
| 2a | LeafNet + Batch Normalization, without augmentation | 0.747 ± 0.025 | 0.503 ± 0.035 |
| 2b | LeafNet + Batch Normalization + augmentation | 0.640 ± 0.025 | 0.609 ± 0.051 |
| 3 | LeafNet + ColorJitter | 0.704 ± 0.025 | 0.640 ± 0.050 |
| 4a | LeafNet + dropout 0.3 | 0.656 ± 0.047 | 0.620 ± 0.076 |
| 4b | LeafNet + dropout 0.5 | 0.581 ± 0.028 | 0.578 ± 0.091 |
| 5 | LeafNet + AdamW | 0.694 ± 0.043 | 0.647 ± 0.055 |
| 6a | MobileNetV3 Small – frozen backbone + augmentation | 0.863 ± 0.011 | 0.837 ± 0.003 |
| 6b | MobileNetV3 Small – fine-tuning | 0.742 ± 0.023 | 0.790 ± 0.023 |
| 6c | MobileNetV3 Small – frozen backbone, without augmentation | 0.823 ± 0.000 | 0.810 ± 0.011 |
| 6d | MobileNetV3 Small – fine-tuning, learning rate 1e-4 | 0.734 ± 0.011 | 0.778 ± 0.006 |
| 7a | LeafNet + cosine scheduler, 30 epochs | 0.694 ± 0.023 | 0.641 ± 0.011 |
| 7b | MobileNetV3 Small – frozen backbone + cosine scheduler | 0.863 ± 0.011 | 0.841 ± 0.003 |
| 8a | LeafNet + Global Average Pooling | 0.522 ± 0.089 | 0.531 ± 0.073 |
| 8b | LeafNet + label smoothing 0.1 | 0.694 ± 0.016 | 0.636 ± 0.041 |
| 9 | Final model – MobileNetV3 Small, frozen backbone + augmentation | 0.871 | 0.831 |

## 4. Evaluation of the experiments

The most striking improvement for the original LeafNet architecture brought the geometric
augmentation. The validation accuracy grew only slightly, from 0.672 to 0.694, but the accuracy
on the camera-like set rose from 0.406 to 0.648. That suggests that the augmentation helped the
model to handle changes in the position and orientation of the leaf better.

Batch Normalization without augmentation reached a rather good validation accuracy of 0.747,
but on the camera-like set it reached only 0.503. A high validation accuracy on its own
therefore did not mean equally good robustness to changes of the image. The combination of
Batch Normalization and augmentation did not bring the expected improvement in this setting.

ColorJitter, AdamW and label smoothing produced results close to the augmented LeafNet, but
they did not prove a clear benefit over the simpler solution. Higher values of dropout made the
results worse, in particular the value 0.5. Global Average Pooling in the tested configuration
clearly reduced the accuracy, which is why I did not use it in the final model. The cosine
scheduler brought no convincing improvement for LeafNet.

The best results were achieved by the pretrained MobileNetV3 Small with most of the network
frozen and only the classification head trained. The variant with augmentation reached in
experiment 6a a validation accuracy of 0.863 and a camera-like accuracy of 0.837. The final
experiment 9 reached 0.871 on the validation set and 0.831 on the camera-like set. The
improvement over LeafNet is substantial in both evaluations.

Fine-tuning of the pretrained weights made the results worse in my experiments. A probable
explanation is the small size of the training set: by further training, the useful features
learned on a large dataset could have been partially damaged. For this reason I preferred the
frozen backbone and training only the classification head.

## 5. Final model and export

As the final solution I chose MobileNetV3 Small with pretrained frozen weights, a modified
classification head and data augmentation. The model has approximately 4,100 trainable
parameters and the exported ONNX file is about 6.37 MB in size. For comparison, the original
LeafNet had approximately 208,292 trainable parameters and an exported ONNX file of about
0.85 MB. A smaller number of trainable parameters therefore does not automatically mean a
smaller resulting file, because MobileNet also contains frozen pretrained weights.

After the export to ONNX I compared the outputs of the model in PyTorch and in ONNX Runtime.
The maximum difference between the compared logits was approximately 9.1 × 10⁻⁶, which
confirms that the export preserved the computations of the model with a very small numerical
deviation.

I also tried this exported model outside the notebook. The course provides a public camera test
into which you can load your own ONNX model; I loaded `leaf_model_task4.onnx` there and used it
on the phone with leaves of all four classes, two samples per class. All eight predictions were
correct and the probability of the right class was between 90 % and 98 % every time. Eight
photographs are a demonstration, not a measurement — the number I rely on stays the accuracy of
0.831 on the 248-image camera-like set — but the test shows that the exported file, its
preprocessing and the order of the classes are correct in a different program as well.

## 6. Limitations and reliability of the results

The training and validation sets are rather small. The camera-like set was created by
transforming the original validation images, and so it does not represent independent testing
on new photographs from various phones and environments. The results should therefore be
understood as a comparison of experimental settings within the available data, not as a
guarantee of the same accuracy in every real situation.

The number of repetitions was not the same for all experiments. Most variants of LeafNet were
evaluated in three runs, for some variants of MobileNetV3 Small two runs were available, and
the final experiment 9 had a single run. Smaller differences are therefore not, in my view,
clear evidence of an improvement. The most convincing is the difference between the baseline
LeafNet and the frozen pretrained MobileNetV3 Small, which showed up on both the validation and
the camera-like set.

Two measurements of the variability tell me how small is too small. The first one is the size
of the sets themselves: the validation set has 62 images, so
**one single image is worth 1.61 percentage points** of accuracy, and the camera-like set has
248 images, where one image is worth 0.40 points. A difference of one or two images is therefore
not yet a difference at all, which is also why every row of the table carries the spread over
the seeds next to the mean.

The second measurement comes from repeating the whole set of experiments. The sequence was
measured twice from the beginning, with the same seeds, the same training and validation images
and the same camera-like set. The means of the rows then moved between the two measurements by
**0.5 percentage points in the median** on both sets; the largest move was **8.6 points** on the
validation set and 5.5 points on the camera-like set, and both belonged to the same row, the
dropout with probability 0.5, which fell from 0.667 to 0.581. Three of the fourteen compared
rows came back identical on both sets and they are exactly the three rows with the frozen
MobileNet backbone (6a, 6c, 7b).

I read the table with these numbers in mind. The rows whose difference against the row they are
compared with is about one point or less — colour jitter, AdamW, label smoothing and the cosine
schedule — are within this variability, and I call them no difference. The improvement of the
frozen MobileNet over the baseline, +19 points of validation and +43 points of camera-like
accuracy, is by an order of magnitude larger than the variability of a repeated measurement.

## 7. Conclusion

The experiments showed that with a small set of images it is advantageous to use a pretrained
model instead of training the whole convolutional network from scratch. The geometric
augmentation helped the original LeafNet considerably when classifying modified images, but the
best overall results were achieved by the frozen MobileNetV3 Small with augmentation. Neither
fine-tuning of the whole network nor the other tested modifications brought a reliable
improvement in my setting.

I therefore selected as the final solution the frozen MobileNetV3 Small with a trained
classification head and augmentation. This solution reached an accuracy of 0.871 on the
validation set and 0.831 on the camera-like set.
