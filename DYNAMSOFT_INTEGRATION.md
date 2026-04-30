
# Dynamsoft Barcode Reader Integration

This module integrates the Dynamsoft Barcode Reader Bundle into the Library Automation System for high-accuracy (99%) barcode decoding.

## Setup Instructions

### 1. Prerequisites
- Python 3.10+
- `dynamsoft-barcode-reader-bundle` package installed.

### 2. Installation
Run the following commands:
```bash
pip install dynamsoft-barcode-reader-bundle
```

### 3. Environment Configuration
Set the `DYNAMSOFT_LICENSE_KEY` environment variable.
Example (Windows PowerShell):
```powershell
$env:DYNAMSOFT_LICENSE_KEY="YOUR_LICENSE_KEY_HERE"
```
Or for temporary testing, the code falls back to a trial key.

## Usage
Import the `barcode_engine` service:
```python
from app.services.barcode_engine import decode_barcode

image_path = "path/to/image.jpg"
result = decode_barcode(image_path)

if result:
    print(f"Decoded: {result}")
else:
    print("No barcode found.")
```

## Performance & Optimization Notes
- **Singleton Initialization**: The `BarcodeEngine` uses a singleton pattern to initialize the license and router only once, avoiding overhead on repeated calls.
- **Resource Management**: The `CaptureVisionRouter` instance is reused.
- **Error Handling**: Handles missing license, file not found, and decoding errors gracefully.
- **Image Formats**: Supports PNG, JPG, PDF, etc.

## Integration Strategy (Hybrid with OCR)
For maximal reliability:
1. Try **Dynamsoft Barcode Reader** first (Primary).
2. If fails, try **Start Barcode Scanner (ZBar/Pyzbar)** (Secondary/Free).
3. If both fail, use **OCR (Tesseract)** to extract ISBN text patterns.
4. Use a **Confidence Score**:
   - Dynamsoft Barcode: 0.99
   - Pyzbar: 0.90
   - OCR Pattern Match: 0.70

## Testing
Run the example script:
```bash
python example.py <path_to_image>
```
If no path is provided, it attempts to use a sample image from the `external/dynamsoft_samples_v2/Images` directory.
