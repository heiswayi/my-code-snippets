Script to unzip all the zipped files recursively in a target folder. The original file will be deleted once the unzipping is successful.

```python
import os
import zipfile
import sys

def unzip_all(base_dir):
    for root, dirs, files in os.walk(base_dir):
        for file in files:
            if file.endswith('.zip'):
                file_path = os.path.join(root, file)
                extract_dir = os.path.join(root, os.path.splitext(file)[0])
                with zipfile.ZipFile(file_path, 'r') as zip_ref:
                    zip_ref.extractall(extract_dir)
                os.remove(file_path)
                print(f'Unzipped: {file_path} to {extract_dir}')
    print("Unzipping completed")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python unzip_script.py <path_to_base_directory>")
        sys.exit(1)
    
    base_directory = sys.argv[1]
    unzip_all(base_directory)
```