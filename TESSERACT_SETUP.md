
# Tesseract OCR Installation Guide for Windows

This project uses Tesseract OCR as a fallback mechanism for book identification.

## Prerequisites
1. **Python Modules**: `pytesseract` (already installed via pip).
2. **Tesseract Engine**: You must have the Tesseract OCR engine installed on your Windows machine.

## Installation Steps
1. Download the Tesseract installer for Windows from: https://github.com/UB-Mannheim/tesseract/wiki
2. Run the installer.
3. Keep note of the installation path (Default: `C:\Program Files\Tesseract-OCR`).
4. **Important**: Add Tesseract to your PATH environment variable.
   - Search "Edit the system environment variables" in Windows Start.
   - Click "Environment Variables".
   - Under "System variables", select `Path` -> `Edit`.
   - Add `C:\Program Files\Tesseract-OCR`.

## Verify Installation
Open a new terminal and run:
```powershell
tesseract --version
```
If this command works, Tesseract is correctly installed.

## Code Configuration
The `hybrid_book_identifier.py` script automatically checks for `C:\Program Files\Tesseract-OCR\tesseract.exe`.
If you installed it elsewhere, you may need to update the path or set the `TESSDATA_PREFIX` environment variable.
