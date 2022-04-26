## Train

### 1. Setup

```
$ pip install -qr requirements.txt  # install dependencies
$ python3 setup.py
```

Create a folder, such as `training-data/`, stored in the top level directory.

The folder structure should resemble:
```
├── images
|   ├── train
|   └── val
└── labels
    ├── train
    └── val
```

The `images/` folder should contain training images (used to train the dataset) and validation images
(used to compare the model's result with the expected result).

The `labels/` folder should correspond to the `images/` folder. Rather than containing images, it should
contain corresponding `.txt` files listing all objects (all occurrences of each class).

You can generate these files with tools such as [labelImg](https://github.com/tzutalin/labelImg) or
[MakeSense](https://www.makesense.ai).

The output for each `.txt` file should look like the following:

```
0 0.268519 0.565278 0.488889 0.513889
0 0.754630 0.557639 0.490741 0.526389
```

The first number references the index of that class, while the other numbers represent the coordinates
and size of each object.

Finally, edit `data/holic.yaml` to set the paths and classes for the above.

### 2. Train

```
python3 train.py --img 640 --batch 16 --epochs 12 --data holic.yaml --weights yolov5s.pt --cache
```

If memory runs out, try reducing the batch size from 16.

Training will display something like following:

```
     Epoch   gpu_mem       box       obj       cls    labels  img_size
     11/11        0G   0.08026   0.06991   0.02252        96       640: 100%|████| 4/4 [01:32<00:00, 23.03s/it]
               Class     Images     Labels          P          R     mAP@.5 mAP@.5:.95: 100%|█| 1/1 [00:11<00:0
                 all         16         44      0.251      0.227      0.174     0.0428
```

The `P` number (`0.251`) means accuracy was 25.1%. The closer to 1.0, the better. The more epoch cycles, the higher the number will be.

Note: Actual training will typically require somewhere between 300 - 1,000 epochs, depending on the dataset.

For Pokémon cards, 200+ epochs seemed to be the sweet spot for achieving consistent/similar 99.0~99.5% accuracy.

[Training tips](https://github.com/ultralytics/yolov5/wiki/Tips-for-Best-Training-Results)

#### Resuming training (if escaped)

```
python3 train.py --resume runs/train/REPLACEME/weights/last.pt
```

#### Train on top of existing model

Replace `--weights` with the best (or latest) training data.

```
python3 train.py --img 640 --batch 16 --epochs 200 --data holic.yaml --weights runs/train/REPLACEME/weights/last.pt --cache
```

### 3. Review

These lines appear when training has finished:

```
Optimizer stripped from runs/train/exp2/weights/last.pt, 14.4MB
Optimizer stripped from runs/train/exp2/weights/best.pt, 14.4MB
Results saved to runs/train/exp2
```

See the above `runs/train/` folder above to review the output.

#### Metrics

https://pro.arcgis.com/en/pro-app/latest/tool-reference/image-analyst/how-compute-accuracy-for-object-detection-works.htm

Precision

> Precision is the ratio of the number of true positives to the total number of positive predictions. For example, if the model detected 100 trees, and 90 were correct, the precision is 90 percent.

Recall

> Recall is the ratio of the number of true positives to the total number of actual (relevant) objects. For example, if the model correctly detects 75 trees in an image, and there are actually 100 trees in the image, the recall is 75 percent.

F1 score

> The F1 score is a weighted average of the precision and recall. Values range from 0 to 1, where 1 means highest accuracy.

### 4. Detect objects in new images

Run the following to use a trained model to find objects in a new image:

```
python3 detect.py --weights runs/train/REPLACEME/weights/last.pt --img 640 --conf 0.25 --source data/images/
```

Replace `best.pt` with a `*.pt` file (displayed after training has finished) to use as `--weights`.

`--conf` determines the minimum "confidence" required for a match.
