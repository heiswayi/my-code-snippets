Script to get correct movie data. Here is what it does:

- Scan the target folder recursively
- Extract and clean movie file name
- Use Google Search to get the correct movie name and movie ID
- Use omdbapi.com to get the movie data based on the movie ID found from Google Search
- Save the correct movie name and year into the `scanned_movies.txt` file

```bash
pip install beautifulsoup4
pip install google
```

```python
import os
import re
import requests
from googlesearch import search

OMDB_API_KEY = 'API_KEY_HERE'
OMDB_URL = 'http://www.omdbapi.com/'

def get_movie_data(movie_id):
    params = {
        'i': movie_id,
        'apikey': OMDB_API_KEY
    }
    response = requests.get(OMDB_URL, params=params)
    if response.status_code == 200:
        return response.json()
    else:
        return None

def clean_movie_name(filename):
    filename = re.sub(r'[\W_]+', ' ', filename)
    filename = filename.strip()
    return filename

def remove_extension(filename):
    return os.path.splitext(filename)[0]

def find_correct_movie_name(messed_up_name):
    query = f"{messed_up_name} movie"
    search_results = search(query, num=5, stop=5, pause=2)  # adjust num and stop parameters as needed
    for result in search_results:
        if "imdb.com/title/" in result:
            return result.split('/')[-2].replace('_', ' ')

    return None

def scan_movies(target_folder, extensions):
    scanned_files = set()
    
    with open("scanned_movies.txt", "w") as output_file:
        for root, dirs, files in os.walk(target_folder):
            for file in files:
                if file.lower().endswith(tuple(extensions)):
                    file_path = os.path.join(root, file)
                    if file not in scanned_files:
                        scanned_files.add(file)
                        movie_name = remove_extension(file)
                        movie_name = clean_movie_name(movie_name)
                        correct_name = find_correct_movie_name(movie_name)
                        if correct_name:
                            movie_data = get_movie_data(correct_name)
                            if movie_data and movie_data.get('Response') == 'True':
                                new_name = f"{movie_data['Title']} ({movie_data['Year']})"
                                output_file.write(f"{new_name}\n")
                                print(f"{new_name}")
                        else:
                            output_file.write(f"{movie_name}\n")
                            print(f"{movie_name}")

target_folder = "E:\\Entertainment\\Movies"
extensions = (".mp4", ".mkv", ".avi")  # Add or remove extensions as needed
scan_movies(target_folder, extensions)
```