Script to delete all empty folders recursively in a target folder.

Using `argparse` instead of `sys.argv`.

```python
import os
import argparse

def delete_empty_folders(target_folder):
    for foldername, subfolders, filenames in os.walk(target_folder, topdown=False):
        if not subfolders and not filenames:
            os.rmdir(foldername)
            print(f"Deleted: {foldername}")
    print("Done!")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Delete empty folders in a directory')
    parser.add_argument('--directory', type=str, required=True, help='Directory to search in')
    args = parser.parse_args()
    delete_empty_folders(args.directory)
```