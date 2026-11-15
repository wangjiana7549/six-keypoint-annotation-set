Six-Keypoint Part Annotations for MAR20,FAIR1M-1.0 and MTARSI
Six aircraft-relative semantic keypoints annotated on top of three public fine-grained
aircraft benchmarks, released with the paper SSP-Det: Sparse Semantic Part Priors for
Generalized Few-Shot Aircraft Detection in Remote-Sensing Images and PAP-Det: Part-Attribute Prototypes for Generalized Few-Shot and Class-Incremental Fine-Grained Aircraft Detection in Remote Sensing Images.
This repository contains annotations only. It does not redistribute the source imagery —
MAR20 ,FAIR1M-1.0 and MTARSI must be obtained from their original providers (see Source data).
---
Keypoint definition
Left and right are defined in an aircraft-relative coordinate system, i.e. from the
pilot's perspective, not in image coordinates. Aircraft appear at arbitrary orientations in
overhead imagery, so this convention is what keeps part identity stable across instances.
Index	Name	Abbrev.	Description
0	Nose	Nose	Foremost point of the fuselage
1	Left wing tip	L Wing	Port-side wing tip
2	Left horizontal-tail tip	L Tail	Port-side horizontal stabilizer tip
3	Vertical-tail tip	V Tail	Topmost point of the vertical stabilizer
4	Right horizontal-tail tip	R Tail	Starboard-side horizontal stabilizer tip
5	Right wing tip	R Wing	Starboard-side wing tip
Indices 0 and 3 lie on the longitudinal axis; indices 1/5 and 2/4 form bilateral pairs.
Horizontal-flip handling
Flipping an image moves a port-side wing to the right of the frame. Transforming coordinates
alone would therefore assign contradictory bilateral labels across training samples. When you
apply a horizontal flip, permute the keypoint indices as well:
```python
PART_HFLIP_INDICES = [0, 5, 4, 3, 2, 1]   # 1<->5, 2<->4; 0 and 3 unchanged
```
Coordinates use the floating-point convention `x' = W - x`, `y' = y` for an image of width `W`
(not the pixel-index form `W - 1 - x`).
---
Statistics
MAR20
	Images	Instances	Fully annotated instances	Keypoints
Train	1,331	7,870	4,485 (56.99%)	26,910
Test	2,511	14,471	—	—
Total	3,842	22,341	4,485	26,910
MTARSI
	Images	Share
Total in directory	8,779	100%
With usable JSON	8,442	96.16%
With complete six-point annotations	8,239	93.85%
Missing JSON	337	3.84%
Incomplete keypoints	203	2.31%
---
Annotation protocol
Annotations are sparse at the instance level but complete per annotated instance: an
instance is labelled only when all six semantic parts are unambiguously identifiable, and it
then receives all six keypoints. Instances that are too small, heavily occluded, truncated by
the image boundary, or otherwise ambiguous are left unlabelled — they keep their original
bounding-box and class labels and simply contribute no part supervision.
All six keypoints carry consistent structural meaning across the 20 aircraft classes.
Annotation was performed with the VGG Image Annotator (VIA).
---
Repository layout
Instances are cropped from the original MAR20 images and stored in per-class folders:
```
.
├── mar20/
│   ├── crops/
│   │   ├── A1/
│   │   │   ├── A1_00001_1_1.jpg
│   │   │   ├── A1_00001_3_2.jpg
│   │   │   └── ...
│   │   ├── A2/
│   │   └── ...            # 20 class folders
│   └── keypoints/         # six-keypoint annotations
├── mtarsi/
│   └── keypoints/
├── tools/
│   └── visualize.py
└── README.md
```
Crop naming convention
```
{class}_{source_image}_{object_index}_{index_within_class}.jpg
```
Field	Meaning
`class`	Aircraft type (MAR20 class label, e.g. `A16`)
`source_image`	Name of the original MAR20 image the crop was taken from
`object_index`	Position of the corresponding `<object>` entry in the source XML, 1-based
`index_within_class`	Running count of crops saved into that class folder, 1-based
`source_image` + `object_index` together identify exactly one box in the original MAR20
annotation, so every crop can be mapped back to its source bounding box without ambiguity.
`index_within_class` is a bookkeeping counter only and carries no semantics.
Example: `A16_00042_3_117.jpg` is the 3rd `<object>` in the XML of source image `00042`,
class `A16`, and the 117th crop stored in the `A16/` folder.
Coverage
The MAR20 training split contains 1,331 images and 7,870 instances, and a crop was
saved for every instance. Six-keypoint annotations exist for the 4,485 instances whose
parts were all unambiguously identifiable (56.99%); the remaining crops carry only their
original box and class labels.
---
Annotation format
<!-- TODO: replace with a real excerpt from your files, and CONFIRM the coordinate frame -->
<!-- IMPORTANT: state explicitly whether `keypoints` are given in CROP pixel coordinates
     (origin at the crop's top-left) or in ORIGINAL-IMAGE coordinates. Users cannot infer
     this, and getting it wrong silently produces garbage. If crop coordinates, also ship
```json
{
  "file_name": "A16_00042_3_117.jpg",
  "source_image": "00042",
  "object_index": 3,
  "category": "A16",
  "bbox": [x1, y1, x2, y2],
  "keypoints": [
    [x0, y0, v0],
    [x1, y1, v1],
    [x2, y2, v2],
    [x3, y3, v3],
    [x4, y4, v4],
    [x5, y5, v5]
  ]
}
```
`keypoints` is ordered by the index table above. The third element of each triplet is a
visibility flag; only visible keypoints contribute to the part-estimation loss in SSP-Det.
---
Quick start
```python
import json

with open("mar20/train_keypoints.json") as f:
    anns = json.load(f)

PART_NAMES = ["Nose", "L Wing", "L Tail", "V Tail", "R Tail", "R Wing"]

a = anns[0]
for name, (x, y, v) in zip(PART_NAMES, a["keypoints"]):
    print(f"{name:8s} ({x:.1f}, {y:.1f}) visible={bool(v)}")
```
---
Source data
Dataset	Reference
MAR20	Yu, W.; et al. MAR20: A benchmark for military aircraft recognition in remote sensing images. National Remote Sensing Bulletin, 2023.
MTARSI	Wu, Z.; et al. Multi-Type Aircraft of Remote Sensing Images (MTARSI), 2020.
Please obtain the imagery from the original sources and comply with their respective terms.
---
License
<!-- TODO: choose a license and add the matching LICENSE file. CC BY 4.0 is a common
The annotations in this repository are released under [TODO]. The underlying images remain
subject to the licenses of MAR20 and MTARSI.
---
Citation
<!-- TODO: update once the paper is published -->
```bibtex
@article{Wang2026SSPDet,
  author  = {Wang, Jian and Zhao, Xinbo and Shao, Jiang and Zou, Xiaochun},
  title   = {{SSP-Det}: Sparse Semantic Part Priors for Generalized Few-Shot Aircraft
             Detection in Remote-Sensing Images},
  journal = {Remote Sensing},
  year    = {2026}
}
```
Contact
jianwang@mail.nwpu.edu.cn (J.W.);
