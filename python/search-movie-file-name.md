Script to search for the movie file name that contains a particular `keyword`, and print its file location.

```python
import os
import sys

def search_movies(root_dir, keyword):
    keyword_lower = keyword.lower()
    for dirpath, _, filenames in os.walk(root_dir):
        for filename in filenames:
            if keyword_lower in filename.lower() and filename.lower().endswith(('.mp4', '.mkv', '.avi', '.mov')):
                print(os.path.join(dirpath, filename))

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: python search_movies.py <directory_path> <keyword>")
        sys.exit(1)

    root_directory = sys.argv[1]
    search_keyword = sys.argv[2]

    search_movies(root_directory, search_keyword)
```