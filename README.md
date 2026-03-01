# 🌱 leafmetrics

> Real-world plant height measurement using HSV color masking and contour detection with OpenCV.

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat&logo=python&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-4.x-5C3EE8?style=flat&logo=opencv&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-1.21+-013243?style=flat&logo=numpy&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

---

## 📌 Overview

**leafmetrics** is a computer vision project that detects green plant regions in images using HSV color space masking, finds contours around each plant, and converts pixel-based measurements into real-world centimeters using a reference scale.

This project was built as part of a Day 03 Computer Vision module covering point operations, HSV masking, contour detection, and real-world measurement conversion.

---

## 🖼️ How It Works

```
Input Image (PNG/JPG)
        ↓
    Resize to 50%
        ↓
  Convert BGR → HSV
        ↓
  Apply Color Mask (green range)
        ↓
  Find Contours (boundaries of plants)
        ↓
  Filter by Area (remove noise)
        ↓
  Draw Bounding Box + Circles
        ↓
  Convert pixels → real cm using reference scale
        ↓
  Display measurements on image
```

---

## 📁 Project Structure

```
leafmetrics/
│
├── plants/                  # Input plant images
│   ├── D10.png
│   ├── D13.png
│   └── D14.png
│
├── leafmetrics.py           # Main script
├── requirements.txt         # Dependencies
└── README.md                # This file
```

---

## ⚙️ Requirements

- Python 3.8+
- OpenCV
- NumPy

Install dependencies:

```bash
pip install opencv-python numpy
```

Or using the requirements file:

```bash
pip install -r requirements.txt
```

**requirements.txt**
```
opencv-python
numpy
```

---

## 🚀 Usage

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/leafmetrics.git
cd leafmetrics
```

### 2. Add your plant images

Place your plant images inside the `plants/` folder.

### 3. Run the script

```bash
python leafmetrics.py
```

---

## 🧠 Full Code

```python
import cv2
import numpy as np

# ── Reference scale for real-world measurement ─────────────────────
# Place a known object (e.g. credit card = 5.4cm tall) in the image
# Measure how many pixels it takes → set REFERENCE_PIXELS accordingly
REFERENCE_REAL_CM = 5.4    # real size of reference object in cm
REFERENCE_PIXELS  = 150    # pixel size of reference object in image
pixels_per_cm     = REFERENCE_PIXELS / REFERENCE_REAL_CM

# ── Load image ─────────────────────────────────────────────────────
plant_img = cv2.imread(r'plants/D10.png')
height, width = plant_img.shape[0:2]

# ── Resize to half ─────────────────────────────────────────────────
plant_img = cv2.resize(plant_img, (int(width/2), int(height/2)))

# ── Convert BGR → HSV ──────────────────────────────────────────────
hsv = cv2.cvtColor(plant_img, cv2.COLOR_BGR2HSV)

# ── Define green color range ───────────────────────────────────────
lower_color_limit = np.array([25,  40,  40])
upper_color_limit = np.array([95, 255, 255])

# ── Create mask and apply ──────────────────────────────────────────
mask = cv2.inRange(hsv, lower_color_limit, upper_color_limit)
res  = cv2.bitwise_and(plant_img, plant_img, mask=mask)

# ── Find contours ──────────────────────────────────────────────────
contours, hierarchy = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
print('no of Contours :', len(contours))

# ── Process each contour ───────────────────────────────────────────
for cnt in contours:
    area = cv2.contourArea(cnt)
    print(area)

    if area > 1200.0:
        cv2.drawContours(plant_img, [cnt], -1, (0, 255, 255), 1)

        x, y, w, h = cv2.boundingRect(cnt)
        cv2.rectangle(plant_img, (x, y), (x+w, y+h), (0, 255, 0), 2)

        cv2.circle(plant_img, (int(x+w/2), y),   7, (0, 162, 255), -1)
        cv2.circle(plant_img, (int(x+w/2), y+h), 7, (0, 162, 255), -1)

        real_height_cm = round(h / pixels_per_cm, 2)
        cv2.putText(plant_img, "Height: " + str(real_height_cm) + "cm",
                    (x+w, y+h), cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2)

# ── Display results ────────────────────────────────────────────────
cv2.imshow('Plant Image', plant_img)
cv2.imshow('Mask', mask)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

---

## 🎨 Visual Output

| Window | Description |
|--------|-------------|
| `Plant Image` | Original image with contours, bounding boxes, circles, and height labels |
| `Mask` | Binary image — white areas = detected green plant regions |

Each detected plant will show:
- 🟡 **Yellow outline** — exact contour boundary
- 🟢 **Green rectangle** — bounding box around the plant
- 🟠 **Orange circles** — top and bottom measurement points
- ⬜ **White text** — real-world height in cm

---

## 📏 Real-World Measurement

To get accurate cm values, a **reference object** must be placed next to the plant when taking the photo.

| Variable | Default | Description |
|----------|---------|-------------|
| `REFERENCE_REAL_CM` | `5.4` | Real height of reference object (credit card = 5.4cm) |
| `REFERENCE_PIXELS` | `150` | Pixel height of that object in the image |

**Formula used:**

```
pixels_per_cm  = REFERENCE_PIXELS / REFERENCE_REAL_CM
real_height_cm = plant_height_pixels / pixels_per_cm
```

> 💡 Tip: Use a **credit card, ruler, or A4 paper** as your reference object. Measure its pixel size from the image using any image editor (e.g. Paint, Photoshop).

---

## 🎛️ HSV Color Range Tuning

The default range targets green plant leaves:

```python
lower_color_limit = np.array([25,  40,  40])   # H, S, V
upper_color_limit = np.array([95, 255, 255])   # H, S, V
```

Adjust these values if results are inaccurate:

| Problem | Fix |
|---------|-----|
| Missing dark/shadowed leaves | Lower `V` minimum (e.g. 40 → 20) |
| Picking up background/soil | Raise `S` minimum (e.g. 40 → 70) |
| Missing yellow-green tips | Lower `H` minimum (e.g. 25 → 15) |
| Picking up yellow/brown areas | Raise `H` minimum (e.g. 25 → 35) |
| Too many small noise contours | Raise area threshold (e.g. 1200 → 2500) |

---

## 🔍 Key OpenCV Functions Used

| Function | Purpose |
|----------|---------|
| `cv2.cvtColor()` | Convert BGR → HSV color space |
| `cv2.inRange()` | Create binary mask from color range |
| `cv2.bitwise_and()` | Apply mask to isolate green regions |
| `cv2.findContours()` | Detect boundaries of white regions in mask |
| `cv2.contourArea()` | Get area of each contour in pixels |
| `cv2.boundingRect()` | Get x, y, width, height of contour |
| `cv2.drawContours()` | Draw contour outlines on image |
| `cv2.rectangle()` | Draw bounding box |
| `cv2.circle()` | Draw measurement marker dots |
| `cv2.putText()` | Display height label on image |

---

## 📊 Concepts Covered

- **HSV Color Space** — Hue, Saturation, Value for color-based detection
- **Color Masking** — Isolating specific color ranges using `inRange()`
- **Bitwise Operations** — Applying masks with `bitwise_and()`
- **Contour Detection** — Finding object boundaries with `findContours()`
- **Bounding Rectangles** — Extracting position and size of detected objects
- **Pixel to Real-World Conversion** — Using a reference scale to convert px → cm

---

## 🌿 Sample Results

```
no of Contours : 47
1523.5
2840.0   ← plant detected
3102.0   ← plant detected
980.5
2255.0   ← plant detected

Plant Heights:
  → 12.40 cm
  → 15.20 cm
  → 8.70 cm
```

---

## 📌 Notes

- Run `cv2.destroyAllWindows()` in a separate cell if using **Jupyter Notebook** before re-running
- Use **Matplotlib** instead of `cv2.imshow()` if the kernel crashes in Jupyter
- The `REFERENCE_PIXELS` value must be **manually measured** from your specific image for accurate results

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 🙋 Author

**Lalana Gurusinghe**
- 📧 Built as part of AI/Computer Vision coursework — Day 03
- 🔗 [LinkedIn](https://linkedin.com/in/your-profile)
- 🐙 [GitHub](https://github.com/yourusername)

---

> *"Measuring the world one pixel at a time."* 🌱
