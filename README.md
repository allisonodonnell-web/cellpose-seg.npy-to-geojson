# Cellpose-seg.npy-to-geojson
Convert Cellpose segmentation outputs (_seg.npy) to GeoJSON for QuPath

# Set up a folder 
```python
mkdir cellpose_geojson
cd cellpose_geojson     ## -> Put your _seg.npy file into this folder
```

# Check it is there 
```
ls 
```

# Create a python environment 
```
conda create -n cp_geojson python=3.10 -y
conda activate cp_geojson
```

# Install required packages 
```
pip install numpy shapely scikit-image
```

# Confirm installs
```
python -c "import numpy, shapely, skimage; print('OK')"
```

# Create the conversion script 
```
nano seg_npy_to_geojson.py
```

# Paste everything below 
```python
import numpy as np
import json
from skimage import measure
from shapely.geometry import Polygon, mapping

INPUT_NPY = "your_file_seg.npy"     # CHANGE THIS
OUTPUT_GEOJSON = "segmentation.geojson"
MIN_AREA = 100                      # increase if QuPath is slow
SIMPLIFY = 1.0                      # set 0.5–2.0 if needed

data = np.load(INPUT_NPY, allow_pickle=True).item()
mask = data["masks"]

features = []

for label in np.unique(mask):
    if label == 0:
        continue

    binary = mask == label
    contours = measure.find_contours(binary, 0.5)

    for c in contours:
        coords = c[:, ::-1]  # (row,col) → (x,y)
        poly = Polygon(coords)

        if not poly.is_valid:
            poly = poly.buffer(0)

        if poly.area < MIN_AREA:
            continue

        if SIMPLIFY > 0:
            poly = poly.simplify(SIMPLIFY, preserve_topology=True)

        features.append({
            "type": "Feature",
            "geometry": mapping(poly),
            "properties": {"cell_id": int(label)}
        })

geojson = {"type": "FeatureCollection", "features": features}

with open(OUTPUT_GEOJSON, "w") as f:
    json.dump(geojson, f)

print(f"Exported {len(features)} objects → {OUTPUT_GEOJSON}")
```

# Save and Exit: Ctrl + O, then enter 
# Ctrl + X

# Edit the file name 
```
nano seg_npy_to_geojson.py
```

# Change: to your actual file name 
```
INPUT_NPY = "your_file_seg.npy"
INPUT_NPY = "sample_seg.npy"
```

# Save and exit again 

# Run the script 
```
python seg_npy_to_geojson.py
```

# If successful you will see 
```
Exported 12345 objects → segmentation.geojson
```

# Check the file exists 
```
ls 
```

## Import into QuPath 
```
File → Import Objects → GeoJSON
```

## Select segmentation.geojson
