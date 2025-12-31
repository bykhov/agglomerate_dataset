# Aluminum Agglomerate Segmentation Dataset

A benchmark dataset for automated detection and characterization of aluminum agglomerates in solid propellant combustion, containing high-speed imagery with instance-level segmentation annotations.

## Overview

This dataset accompanies the paper:

> **Automated Detection and Characterization of Aluminum Agglomerates in Solid Propellant Combustion via Mask R-CNN**


Aluminum agglomeration in solid rocket propellants significantly impacts two-phase flow losses and combustion efficiency. This dataset provides manually annotated high-speed images of burning aluminized propellants, enabling the development and benchmarking of automated agglomerate detection algorithms.

## Dataset Statistics

| Property | Value |
|----------|-------|
| Total Images | 166 |
| Total Annotations | 8,160 |
| Image Resolution | 800 × 480 pixels |
| Image Format | TIFF |
| Annotation Format | COCO JSON |

### Pressure Distribution

| Pressure | Images | Annotations |
|----------|--------|-------------|
| 2 MPa | 86 | 5,830 |
| 3 MPa | 39 | 351 |
| 4 MPa | 41 | 1,979 |

## Experimental Setup

### Imaging Equipment
- **Camera**: Chronos 2.1 high-speed camera
- **Frame Rate**: 5,000–5,400 fps
- **Exposure Time**: 15 μs
- **Spatial Resolution**: 10–13 μm/pixel

### Combustion Chamber
- Nitrogen-pressurized chamber (~1 L volume)
- 25 mm thick glass window for optical access
- Operating pressure range: 2–4 MPa
- Ignition: Resistive filament (10 W supply)

### Propellant Composition

| Material | Function | Particle Size | Mass Fraction |
|----------|----------|---------------|---------------|
| HTPB | Fuel binder | — | 18% |
| IPDI | Cross-linker | — | 2% |
| AP (coarse) | Oxidizer | 400 μm | 52% |
| AP (fine) | Oxidizer | 90 μm | 13% |
| Al (micro) | Metallic fuel | 6 μm | 10% |
| Al (nano) | Metallic fuel | 50–70 nm | 5% |

## Dataset Structure

```
Data/
├── images/
│   ├── 2MPa/
│   │   └── frame_XXXXXX.tiff
│   ├── 3MPa/
│   │   └── frame_XXXXXX.tiff
│   └── 4MPa/
│       └── frame_XXXXXX.tiff
├── annotations.json
└── README.md
```

## Annotation Format

Annotations follow the [COCO format](https://cocodataset.org/#format-data) with instance-level polygonal segmentation masks.

### JSON Structure

```json
{
  "images": [
    {
      "id": 186,
      "width": 800,
      "height": 480,
      "file_name": "frame_000165.tiff",
      "path": "/Data/2MPa/frame_000165.tiff",
      "annotated": true,
      "num_annotations": 82,
      "metadata": {
        "pressure": "2MPa",
        "pressure_mpa": 2
      }
    }
  ],
  "annotations": [
    {
      "id": 4545,
      "image_id": 186,
      "category_id": 2,
      "segmentation": [[x1, y1, x2, y2, ...]],
      "area": 32,
      "bbox": [x, y, width, height],
      "iscrowd": false
    }
  ],
  "categories": [
    {
      "id": 2,
      "name": "Particle",
      "supercategory": ""
    }
  ]
}
```

### Annotation Fields

| Field | Description |
|-------|-------------|
| `segmentation` | List of polygon vertices [x1, y1, x2, y2, ...] |
| `area` | Mask area in pixels² |
| `bbox` | Bounding box [x_min, y_min, width, height] |
| `iscrowd` | Always `false` (individual instances) |

### Image Metadata Fields

| Field | Description |
|-------|-------------|
| `pressure` | Operating pressure ("2MPa", "3MPa", "4MPa") |
| `pressure_mpa` | Numeric pressure value (2, 3, or 4) |
| `num_annotations` | Count of annotations in this image |
| `annotated` | Always `true` for annotated images |

## Usage

### Loading with PyTorch

```python
import json
from PIL import Image
import numpy as np

# Load annotations
with open('annotations.json', 'r') as f:
    coco_data = json.load(f)

# Create image ID to annotations mapping
img_to_anns = {}
for ann in coco_data['annotations']:
    img_id = ann['image_id']
    if img_id not in img_to_anns:
        img_to_anns[img_id] = []
    img_to_anns[img_id].append(ann)

# Load an image and its annotations
img_info = coco_data['images'][0]
image = Image.open(f"images/{img_info['file_name']}")
annotations = img_to_anns[img_info['id']]
```

### Loading with pycocotools

```python
from pycocotools.coco import COCO

coco = COCO('annotations.json')
img_ids = coco.getImgIds()
ann_ids = coco.getAnnIds(imgIds=img_ids[0])
anns = coco.loadAnns(ann_ids)
```

### Filtering by Pressure

```python
# Filter images by pressure condition
pressure_2mpa = [img for img in coco_data['images'] 
                 if img['metadata']['pressure'] == '2MPa']
pressure_3mpa = [img for img in coco_data['images'] 
                 if img['metadata']['pressure'] == '3MPa']
pressure_4mpa = [img for img in coco_data['images'] 
                 if img['metadata']['pressure'] == '4MPa']

print(f"2 MPa: {len(pressure_2mpa)} images")  # 86 images
print(f"3 MPa: {len(pressure_3mpa)} images")  # 39 images
print(f"4 MPa: {len(pressure_4mpa)} images")  # 41 images
```


## Contact

For questions about the dataset, please open an issue in this repository.
