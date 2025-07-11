```python
# NORMALIZATION
import cv2
import numpy as np
from pathlib import Path
from PIL import Image

# === PATHS ===
reference_path = Path(r"C:\Users\DELL\Desktop\NIR IMAGES\sample 7 images\reference images\370nm\image_1.png")
sample_path = Path(r"C:\Users\DELL\Desktop\NIR IMAGES\sample 7 images\sample images\370nm\image_1.png")
output_dir = Path(r"C:\Users\DELL\Desktop\NIR IMAGES\sample 7 images\normalized images\370nm")
output_dir.mkdir(parents=True, exist_ok=True)

# === LOAD REFERENCE IMAGE ===
ref_image_raw = cv2.imread(str(reference_path), cv2.IMREAD_GRAYSCALE)
if ref_image_raw is None:
    raise FileNotFoundError(f"Reference image not found at {reference_path}")
ref_image = ref_image_raw.astype(np.float32)

# === NORMALIZE ONE SAMPLE ===
def normalize_sample(sample_path):
    sample_img = cv2.imread(str(sample_path))
    if sample_img is None:
        print(f"Failed to load: {sample_path.name}")
        return

    gray_sample = cv2.cvtColor(sample_img, cv2.COLOR_RGB2GRAY).astype(np.float32)
    ref_resized = cv2.resize(ref_image, (gray_sample.shape[1], gray_sample.shape[0]))

    normalized = (gray_sample - ref_resized) / (ref_resized + 1e-5)
    normalized = np.clip(normalized, -1, 1)
    normalized = ((normalized + 1) * 127.5).astype(np.uint8)

    output_path = output_dir / sample_path.name
    Image.fromarray(normalized).save(output_path)
    print(f"✅ Saved normalized image: {output_path.name}")

normalize_sample(sample_path)

```
