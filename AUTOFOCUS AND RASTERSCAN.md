```python
import time
from pathlib import Path
import numpy as np
import cv2
from PIL import Image
import matplotlib.pyplot as plt
from openflexure_microscope_client import MicroscopeClient

# Connect to microscope
microscope = MicroscopeClient("192.168.0.111")

# Parameters
coarse_step_size = 200
coarse_num_steps = 3
fine_step_size = 50
fine_num_steps = 2
settle_time = 0.01  # Reduced settle time for speed
patience = 3  # Number of stagnant steps allowed before stopping

def compute_laplacian_variance(image):
    gray = cv2.cvtColor(image, cv2.COLOR_RGB2GRAY)
    return cv2.Laplacian(gray, cv2.CV_64F).var()

def move_microscope_rel(z_delta):
    microscope.move_rel({"x": 0, "y": 0, "z": z_delta})
    time.sleep(settle_time)

def move_microscope_abs(z_target):
    microscope.move({"x": microscope.position['x'], "y": microscope.position['y'], "z": z_target})
    time.sleep(settle_time)

def capture_image():
    return np.array(microscope.grab_image())

def evaluate_focus_direction(step_size, num_steps):
    base_z = microscope.position['z']
    variances = []

    # Move up num_steps, record variances
    for _ in range(num_steps):
        move_microscope_rel(step_size)
        var = compute_laplacian_variance(capture_image())
        variances.append((var, microscope.position['z']))

    # Move back to base
    move_microscope_rel(-step_size * num_steps)

    # Move down num_steps, record variances
    for _ in range(num_steps):
        move_microscope_rel(-step_size)
        var = compute_laplacian_variance(capture_image())
        variances.append((var, microscope.position['z']))

    # Move back to base
    move_microscope_rel(step_size * num_steps)

    # Find max variance position
    best_var, best_z = max(variances, key=lambda x: x[0])

    direction = 1 if best_z > base_z else -1
    return direction

def move_until_focus_drops(step_size, direction, patience=3):
    best_var = -1
    best_z = microscope.position['z']
    best_image = None
    stagnant_steps = 0

    while stagnant_steps < patience:
        move_microscope_rel(direction * step_size)
        image = capture_image()
        var = compute_laplacian_variance(image)

        if var > best_var + 1e-4:
            best_var = var
            best_z = microscope.position['z']
            best_image = image
            stagnant_steps = 0
        else:
            stagnant_steps += 1

    # Return to best focus position
    move_microscope_abs(best_z)
    return best_z, best_var, best_image

def coarse_focus():
    print("Evaluating direction for coarse focus...")
    direction = evaluate_focus_direction(coarse_step_size, coarse_num_steps)
    print(f"Coarse focus direction: {'up' if direction == 1 else 'down'}")
    best_z, best_var, best_image = move_until_focus_drops(coarse_step_size, direction, patience)
    print(f"Coarse focus at Z={best_z} with variance={best_var:.2f}")
    return best_z, best_var, best_image

def fine_focus(coarse_z):
    print("Starting fine focus...")
    move_microscope_abs(coarse_z)
    base_z = coarse_z

    # Scan +/- fine_num_steps * fine_step_size to find better peak
    variances = []
    images = []

    for i in range(-fine_num_steps, fine_num_steps + 1):
        target_z = base_z + i * fine_step_size
        move_microscope_abs(target_z)
        image = capture_image()
        var = compute_laplacian_variance(image)
        variances.append((var, target_z))
        images.append(image)

    best_var, best_z = max(variances, key=lambda x: x[0])
    best_image = images[[z for _, z in variances].index(best_z)]

    # Move to best_z before fine tuning direction
    move_microscope_abs(best_z)

    # Evaluate direction of increasing variance at fine scale
    direction = evaluate_focus_direction(fine_step_size, fine_num_steps)  # just one step each side for fine tuning
    print(f"Fine focus direction: {'up' if direction == 1 else 'down'}")

    # Move until variance drops at fine scale
    best_z, best_var, best_image = move_until_focus_drops(fine_step_size, direction, patience)
    print(f"Fine focus done at Z={best_z} with variance={best_var:.2f}")
    return best_z, best_var, best_image

def autofocus():
    coarse_z, _, coarse_image = coarse_focus()
    move_microscope_abs(coarse_z)
    fine_z, _, fine_image = fine_focus(coarse_z)
    move_microscope_abs(fine_z)
    return fine_z, fine_image

# Output directory
desktop = Path.home() / "Desktop"
output_dir = desktop /  
output_dir.mkdir(parents=True, exist_ok=True)

# Store initial position
starting_pos = microscope.position.copy()

# Raster scan parameters
step_size_x = 800
step_size_y = 800
x_direction = 1
x_steps = 0
max_x_steps = 5

#start_time=time.time()
#total_time=0
# Raster scan and capture 100 images
for i in range(2):
    if i > 0:
        if x_steps < max_x_steps - 1:
            starting_pos['x'] += x_direction * step_size_x
            x_steps += 1
        else:
            starting_pos['y'] += step_size_y
            x_direction *= -1
            x_steps = 0

        print(f"Moving to next position: X={starting_pos['x']}, Y={starting_pos['y']}")
        microscope.move(starting_pos)
        print(f"Position after move: {microscope.position}")

    # Autofocus at current position, returns best Z and focused image
    best_z, focused_image = autofocus()

    # Update Z in starting_pos
    starting_pos['z'] = best_z

    # Save focused image
    image_filename = output_dir / f"image_{i+1}.png"
    Image.fromarray(focused_image).save(image_filename)
    print(f"Saved image {i+1} at {image_filename}")

    # Optional: Display preview (comment out to speed up)
    plt.imshow(focused_image)
    plt.title(f"Image {i+1} | X: {starting_pos['x']}  Y: {starting_pos['y']}  Z: {best_z}")
    plt.axis('off')
    plt.show()
    #end_time=time.time()
    #time_taken=end_time-start_time
    #print(f"time taken to capture and process image { i+1}:{time_taken:.2f} seconds")
    #total_time+=time_taken
# Return to starting position after scanning
microscope.move(starting_pos)
assert microscope.position == starting_pos

print(f"Captured and saved 2 images in '{output_dir}'.")
#average_time=total_time/2
#print(f"average time taken to capture 2 images:{average_time:.2f} seconds")
```
