Script to generate OpenAPI schema (e.g. openapi.json file) from FastAPI code without running the FastAPI app.

```bash
python generate_openapi_schema.py --output openapi.json
```

```python
import argparse
import json
import os
import sys

from myapp.api.v1 import app
from fastapi import FastAPI

# # Extend the sys.path to include the required module
# sys.path.append(
#     os.path.join(os.path.dirname(__file__), "..", "..", "backend", "utils")
# )


if __name__ == "__main__":
    assert isinstance(app, FastAPI), "The app should be an instance of FastAPI."

    parser = argparse.ArgumentParser(description="Generate OpenAPI schema")
    parser.add_argument(
        "--output",
        type=str,
        default="openapi.json",
        help="The output file path for the OpenAPI schema",
    )
    args = parser.parse_args()

    openapi_schema = app.openapi()

    with open(args.output, "w") as file:
        json.dump(openapi_schema, file, indent=2)
```
