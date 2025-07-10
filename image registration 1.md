```python
import os
import numpy as np
from skimage import io, exposure
from skimage.registration import phase_cross_correlation
from skimage.metrics import structural_similarity as ssim
from scipy.ndimage import shift
import matplotlib.pyplot as plt
    
# === Paths ===
main_folder = r"C:\Users\DELL\Desktop\NIR IMAGES\sample 5 images"
input_folder = os.path.join(main_folder, "normalized images")
output_folder = os.path.join(main_folder, "alligned images")
os.makedirs(output_folder, exist_ok=True)

ref_subfolder = "660nm"
ref_filename = "image_1.png"
moving_subfolders = ["405nm", "680nm", "645nm", "430nm", "670nm", "800nm", "850nm"]

# === Load Reference Image ===
ref_path = os.path.join(input_folder, ref_subfolder, ref_filename)
ref_image = io.imread(ref_path).astype(np.float32)

# === Normalize and save reference image ===
ref_norm = np.clip(ref_image, 0, 255)
ref_uint8 = ref_norm.astype(np.uint8)
ref_output_path = os.path.join(output_folder, "660nm_registered.png")
io.imsave(ref_output_path, ref_uint8)
print(f"✅ Reference image saved to: {ref_output_path}")

# === Loop Through and Register ===
for subfolder in moving_subfolders:
    print(f"\n🔄 Registering: {subfolder}")
    moving_path = os.path.join(input_folder, subfolder, "image_1.png")

    if not os.path.exists(moving_path):
        print(f"⚠️ Not found: {moving_path}")
        continue

    # Load and match histogram
    moving_image = io.imread(moving_path).astype(np.float32)
    moving_matched = exposure.match_histograms(moving_image, ref_image)

    # Estimate shift using histogram-matched version
    shift_estimate, error, _ = phase_cross_correlation(ref_image, moving_matched, upsample_factor=10)
    print(f"Estimated shift: {shift_estimate}")

    # Apply shift to the original (unmatched) image to retain original contrast
    registered = shift(moving_image, shift=shift_estimate, mode='constant', cval=0)

    # Normalize for PNG (0–255) and convert to uint8
    registered_norm = np.clip(registered, 0, 255)
    registered_uint8 = registered_norm.astype(np.uint8)

    # Save as PNG
    output_filename = f"{subfolder}_registered.png"
    output_path = os.path.join(output_folder, output_filename)
    io.imsave(output_path, registered_uint8)
    print(f"✅ Saved to: {output_path}")

    # === Compute SSIM ===
    ref_crop = ref_uint8[:registered_uint8.shape[0], :registered_uint8.shape[1]]
    reg_crop = registered_uint8[:ref_crop.shape[0], :ref_crop.shape[1]]
    ssim_score = ssim(ref_crop, reg_crop, data_range=255)
    print(f"🔍 SSIM with {subfolder}: {ssim_score:.4f}")

    # === Optional visualization ===
    plt.figure(figsize=(12, 4))
    plt.subplot(1, 3, 1)
    plt.title("Reference (660nm)")
    plt.imshow(ref_image, cmap='gray')

    plt.subplot(1, 3, 2)
    plt.title(f"Before ({subfolder})")
    plt.imshow(moving_matched, cmap='gray')

    plt.subplot(1, 3, 3)
    plt.title(f"Registered ({subfolder})")
    plt.imshow(registered_uint8, cmap='gray')

    plt.tight_layout()
    plt.show()

```
