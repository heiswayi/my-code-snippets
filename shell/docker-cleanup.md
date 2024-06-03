Script to remove stopped containers and unused images for Docker

`docker_cleanup.sh`

```bash
#!/bin/bash

# Remove stopped containers
echo "Removing non-running containers..."
docker container prune --force >/dev/null 2>&1

# Remove unused images
echo "Removing unused images..."
docker image prune --force >/dev/null 2>&1

echo "Cleanup complete!"
```

```bash
chmod +x docker_cleanup.sh
```