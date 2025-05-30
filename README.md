# samplearea
Python script for finding area of white sample on black background (with noise)

Hi everyone, so for my project I am photographing samples of which I then need to measure the area. The images are on a black background of a white sample (with some gradient in them) and some smaller reflextions from surrounding water. I was thinking something along the lines of the code below but this does not seem to work and I am kinda stuck on why. Bc of large quantity of pictures I thought a script would be useful. The images are in TIFF format.

Code:
import cv2
import numpy as np
import os
import csv

# --- Configuration ---
input_folder = r"C:\Users\joost\OneDrive - TU Eindhoven\YEAR 3\BEP Bachelor End Project (Scriptie)\Data\Keyence\Batch 1\Day 7 exp\TIFF"
output_folder = r"C:\Users\joost\OneDrive\Desktop\AREA TEST"
csv_output_path = r"C:\Users\joost\OneDrive\Desktop\AREA TEST\Area python.xlsx"

os.makedirs(output_folder, exist_ok=True)

# --- CSV Setup ---
with open(csv_output_path, mode='w', newline='') as csv_file:
    writer = csv.writer(csv_file)
    writer.writerow(["Filename", "Largest_Object_Area"])

    # --- Loop through all TIFF files ---
    for filename in os.listdir(input_folder):
        if filename.lower().endswith(".tif"):
            filepath = os.path.join(input_folder, filename)
            image = cv2.imread(filepath, cv2.IMREAD_GRAYSCALE)

            # Threshold (you may need to adjust 200 depending on image contrast)
            _, thresh = cv2.threshold(image, 150, 255, cv2.THRESH_BINARY)

            # Find contours
            contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
            if not contours:
                writer.writerow([filename, 0])
                continue

            # Find the largest contour
            largest = max(contours, key=cv2.contourArea)
            area = cv2.contourArea(largest)

            # Create color overlay
            overlay = cv2.cvtColor(image, cv2.COLOR_GRAY2BGR)
            cv2.drawContours(overlay, [largest], -1, (0, 255, 0), 2)

            # Save overlay image
            output_path = os.path.join(output_folder, f"overlay_{filename}")
            cv2.imwrite(output_path, overlay)

            # Write area to CSV
            writer.writerow([filename, area])

print(f"Done. Results saved to {csv_output_path} and overlays to {output_folder}")
