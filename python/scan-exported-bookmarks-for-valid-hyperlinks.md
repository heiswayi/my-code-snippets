Script to scan all the exported bookmark files to check for only valid hyperlinks.

```python
import os
import re
import requests

def find_hyperlinks(text):
    """Find all hyperlinks in the given text."""
    url_pattern = re.compile(r'http[s]?://\S+')
    return url_pattern.findall(text)

def check_hyperlink(url):
    """Check if the hyperlink is working."""
    try:
        response = requests.head(url, allow_redirects=True, timeout=5)
        return response.status_code == 200
    except requests.RequestException:
        return False

def scan_directory(target_directory, output_file):
    """Scan all files in the target directory recursively, find hyperlinks, and save working ones."""
    link_count = 0
    with open(output_file, 'a') as outfile:
        for root, _, files in os.walk(target_directory):
            for file in files:
                file_path = os.path.join(root, file)
                try:
                    with open(file_path, 'r', encoding='utf-8') as infile:
                        content = infile.read()
                        hyperlinks = find_hyperlinks(content)
                        for link in hyperlinks:
                            link_count += 1
                            print(f"[     ] Checking link {link_count}: {link}")
                            if check_hyperlink(link):
                                print(f"[VALID] Checking link {link_count}: {link}")
                                outfile.write(link + '\n')
                except (UnicodeDecodeError, FileNotFoundError, PermissionError) as e:
                    print(f"Error reading {file_path}: {e}")

if __name__ == "__main__":
    target_directory = "bookmarks"  # Replace with your target directory
    output_file = "valid_hyperlinks.txt"
    scan_directory(target_directory, output_file)
```