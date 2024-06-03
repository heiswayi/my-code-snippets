Script to convert all the `*.webp` files into the `*.jpg` files recursively in a target folder. The original WEBP file will be deleted once the conversion is successful.

```bash
pip install pillow
```

```python
import os
import sys
from PIL import Image

def convert_webp_to_jpg(target_folder):
    for root, _, files in os.walk(target_folder):
        for filename in files:
            if filename.endswith(".webp"):
                webp_path = os.path.join(root, filename)
                jpg_path = os.path.join(root, filename[:-5] + ".jpg")
                
                with Image.open(webp_path) as img:
                    img.convert("RGB").save(jpg_path, "JPEG")
                
                os.remove(webp_path)
                print(f"Converted: {webp_path} to {jpg_path}")
    print("Conversion completed")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python convert_webp_to_jpg.py <target_folder>")
        sys.exit(1)
    
    target_folder = sys.argv[1]
    convert_webp_to_jpg(target_folder)
```