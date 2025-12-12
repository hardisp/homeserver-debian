```sh
#!/usr/bin/env bash
set -euo pipefail

# nextcloud-reset.sh
# Usage:
#   sudo bash nextcloud-reset.sh           # backup, remove all, delete host data, rebuild
#   sudo bash nextcloud-reset.sh --keep-host-data  # backup, remove containers/images, but keep host data

KEEP_HOST_DATA=false
if [ "${1:-}" = "--keep-host-data" ]; then
  KEEP_HOST_DATA=true
fi

# --- CONFIG: sesuaikan bila perlu ---
COMPOSE_FILE="${PWD}/docker-compose.yml"  # pastikan skrip dijalankan di folder yang benar
BACKUP_DIR="/root/nextcloud-backups"
HOST_PATHS_TO_REMOVE=(
  "/mnt/nextcloud-data/cloud-data"
  "/data/volumes/apps/nextcloud/data"
  "/data/volumes/apps/nextcloud/config"
  "/data/volumes/apps/nextcloud/apps"
  "/data/volumes/apps/nextcloud/html"
  "/data/volumes/apps/nextcloud-db/pgdata"
)
# nama container (sesuai docker-compose Anda)
CONTAINERS_TO_REMOVE=(nextcloud nextcloud-db nexcloud-redis)
IMAGES_TO_PRUNE=true   # set false jika tidak ingin menghapus images

# --- end CONFIG ---

echo "Starting Nextcloud reset script..."
echo "Keep host data: ${KEEP_HOST_DATA}"
mkdir -p "${BACKUP_DIR}"

timestamp="$(date +%F_%H%M%S)"
# 1) Stop and remove containers (compose-aware)
if [ -f "${COMPOSE_FILE}" ]; then
  echo "Stopping services via docker compose..."
  docker compose down --rmi local --volumes --remove-orphans || true
else
  echo "docker-compose file not found at ${COMPOSE_FILE}. Will attempt to stop containers by name."
fi

# 2) Stop and remove named containers (extra safety)
for c in "${CONTAINERS_TO_REMOVE[@]}"; do
  if docker ps -a --format '{{.Names}}' | grep -wq "$c"; then
    echo "Stopping container $c..."
    docker stop "$c" || true
    echo "Removing container $c..."
    docker rm -f "$c" || true
  else
    echo "Container $c not present, skipping."
  fi
done

# 3) Backup host bind-mount directories if exist
echo "Creating backups (if directories exist)..."
for p in "${HOST_PATHS_TO_REMOVE[@]}"; do
  if [ -d "$p" ]; then
    tarname="${BACKUP_DIR}/backup_$(basename "$p")_${timestamp}.tgz"
    echo " - backing up $p -> $tarname"
    tar -C "$(dirname "$p")" -czf "$tarname" "$(basename "$p")" || echo "Warning: backup of $p failed"
  else
    echo " - host path $p not found, skipping backup."
  fi
done

# 4) Remove host data directories (if not keeping)
if [ "${KEEP_HOST_DATA}" = false ]; then
  echo "Removing host data directories (if exist)..."
  for p in "${HOST_PATHS_TO_REMOVE[@]}"; do
    if [ -d "$p" ]; then
      echo " - removing $p"
      rm -rf "$p"
    fi
  done
else
  echo "Keeping host data as requested (--keep-host-data)."
fi

# 5) Optionally remove docker images used by the compose
if [ "${IMAGES_TO_PRUNE}" = true ]; then
  echo "Pruning unused docker images..."
  docker image prune -af || true
fi

# 6) Prune unused volumes (careful: will remove dangling volumes)
echo "Pruning dangling docker volumes..."
docker volume prune -f || true

# 7) Pull images and bring up compose (fresh)
if [ -f "${COMPOSE_FILE}" ]; then
  echo "Pulling images and starting fresh containers via docker compose..."
  docker compose pull || true
  docker compose up -d --force-recreate --remove-orphans
  echo "Containers started. Run 'docker compose ps' to verify."
else
  echo "No docker-compose.yml found at ${COMPOSE_FILE}. Stopped after cleanup."
fi

echo "Done. Backups (if any) are in: ${BACKUP_DIR}"
echo "If you kept host data, it was NOT deleted. If you chose default, host data directories listed in script were deleted."
```