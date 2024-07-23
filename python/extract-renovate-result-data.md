This script is used to extract the Renovate logs for the `config` data.

`run_renovate.sh`
```sh
export RENOVATE_LOG_FILE=renovate.log
export RENOVATE_LOG_FILE_LEVEL=debug
export LOG_LEVEL=debug
npx renovate --platform=local --repository-cache=reset
```

`extract_renovate_log.py`
```python
import csv
import json


def extract_config_data(log_content):
    log_lines = log_content.strip().split("\n")
    config_data = []

    for line in log_lines:
        log_entry = json.loads(line)
        if "config" in log_entry:
            config = log_entry["config"]
            if isinstance(config, dict):
                for datasource, data_configs in config.items():
                    if isinstance(data_configs, list):
                        for data_config in data_configs:
                            if isinstance(data_config, dict):
                                for dep in data_config.get("deps", []):
                                    flattened_dep = {
                                        "datasource": datasource,
                                        "depType": dep.get("depType"),
                                        "depName": dep.get("depName"),
                                        "currentValue": dep.get("currentValue"),
                                        "prettyDepType": dep.get("prettyDepType"),
                                        "lockedVersion": dep.get("lockedVersion"),
                                        "currentVersion": dep.get("currentVersion"),
                                        "currentVersionTimestamp": dep.get(
                                            "currentVersionTimestamp"
                                        ),
                                        "isSingleVersion": dep.get("isSingleVersion"),
                                        "fixedVersion": dep.get("fixedVersion"),
                                    }
                                    updates = dep.get("updates", [])
                                    for update in updates:
                                        update_data = flattened_dep.copy()
                                        update_data.update(
                                            {
                                                "updateType": update.get("updateType"),
                                                "newVersion": update.get("newVersion"),
                                                "newValue": update.get("newValue"),
                                                "releaseTimestamp": update.get(
                                                    "releaseTimestamp"
                                                ),
                                                "newMajor": update.get("newMajor"),
                                                "newMinor": update.get("newMinor"),
                                                "newPatch": update.get("newPatch"),
                                                "isRange": update.get("isRange"),
                                                "branchName": update.get("branchName"),
                                            }
                                        )
                                        config_data.append(update_data)
                                    if not updates:
                                        config_data.append(flattened_dep)

    return config_data


def save_to_csv(config_data, filename):
    columns = [
        "datasource",
        "depType",
        "depName",
        "currentValue",
        "prettyDepType",
        "lockedVersion",
        "currentVersion",
        "currentVersionTimestamp",
        "isSingleVersion",
        "fixedVersion",
        "updateType",
        "newVersion",
        "newValue",
        "releaseTimestamp",
        "newMajor",
        "newMinor",
        "newPatch",
        "isRange",
        "branchName",
    ]

    with open(filename, mode="w", newline="") as file:
        writer = csv.DictWriter(file, fieldnames=columns)
        writer.writeheader()
        for data in config_data:
            writer.writerow(data)


csv_filename = "renovate_results.csv"
with open("renovate.log", "r") as file:
    log_content = file.read()
config_data = extract_config_data(log_content)
save_to_csv(config_data, csv_filename)
```
