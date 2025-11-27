# Orthanc Docker (Linux) — Full Guide

Turn-key stack: Orthanc with SQLite, Orthanc Explorer 2 (UI), DICOMweb, OHIF viewer, Python/GDCM/Housekeeper plugins, storage compression enabled, and overwrite of instances allowed.

---

## 1) Prerequisites
- Linux (Fedora/Ubuntu or similar) with Docker and Docker Compose v2 (`docker compose`).
- Ports 8042 (HTTP/REST/UI) and 4242 (DICOM) available.
- Docker daemon running:
  - Linux (systemd): `sudo systemctl start docker`
  - macOS: open Docker Desktop

Check installation:
```bash
docker --version
docker compose version
```
If missing: Fedora `sudo dnf install docker-ce docker-ce-cli containerd.io`; Ubuntu `sudo apt-get install docker-ce docker-ce-cli containerd.io`; then `sudo systemctl enable --now docker`.

---

## 2) What this stack does
- Orthanc container with **SQLite** only (no external DB).
- UI: **Orthanc Explorer 2** at `/ui/app/`.
- **DICOMweb** at `/dicom-web/`.
- **OHIF** viewer at `/ohif/` via DICOMweb.
- Plugins: **Python**, **GDCM**, **Housekeeper**.
- **StorageCompression: true** (compress Orthanc storage blobs).
- **OverwriteInstances: true** (allow replacing same SOP Instance UID).
- Data persisted in `./orthanc-db` (mapped to `/var/lib/orthanc/db`).

---

## 3) File layout
- `docker-compose.yml` — service, volumes, ports.
- `orthanc-config/orthanc.json` — main config (plugins, auth, DICOMweb, OHIF, compression, overwrite).
- `orthanc-python/startup.py` — Python plugin script.
- `orthanc-db/` — DICOM data + SQLite (created on first run).

---

## 4) Start/stop
```bash
# start in background
docker compose up -d

# status
docker compose ps

# live logs
docker compose logs -f orthanc

# stop and remove container (keep data)
docker compose down

# stop and delete data too
docker compose down -v
```

---

## 5) URLs and services
- Orthanc Explorer 2: http://localhost:8042/ui/app/
- OHIF viewer: http://localhost:8042/ohif/
- DICOMweb: http://localhost:8042/dicom-web/studies
- DICOM port: 4242 — AE Title: `ORTHANC`
  - If you prefer host port 104, change mapping to `104:4242` in `docker-compose.yml`, then `docker compose down` + `docker compose up -d`.

---

## 6) Default credentials (change them)
- User: `admin`
- Password: `admin`
Edit `orthanc-config/orthanc.json` (`RegisteredUsers`) and restart.

---

## 7) Security
- `RemoteAccessAllowed=true` is for testing; in production set to `false` or secure via firewall/reverse proxy.
- Change the default password before exposing.

---

## 8) Common tweaks
- AE Title: `DicomAet` in `orthanc-config/orthanc.json`.
- Remote modalities:
```json
"DicomModalities": {
  "REMOTE_SCANNER": ["REMOTE_SCANNER", "192.168.0.50", 104]
}
```
- Overwrite: already `true` to accept re-import of the same SOP Instance UID (e.g., recompressed JPEG2000). Set to `false` to block replacements.
- Storage compression: already `true` to save disk space (uses more CPU).
- After changes: `docker compose up -d` to reload.

---

## 9) Quick tests
- System health:
```bash
curl -u admin:admin http://localhost:8042/system
```
- List studies (DICOMweb):
```bash
curl -u admin:admin http://localhost:8042/dicom-web/studies
```
- Send DICOM (dcmtk):
```bash
storescu -aec ORTHANC localhost 4242 *.dcm
```
If you mapped host port 104, send to `localhost 104`.

---

## 10) Plugins included
- Orthanc Explorer 2 (default UI)
- DICOMweb
- OHIF (DICOMweb data source)
- Python (loads `orthanc-python/startup.py`)
- GDCM (JPEG2000, etc.)
- Housekeeper

---

## 11) Config highlights
- Storage: `/var/lib/orthanc/db` (SQLite).
- Plugins path: `/usr/share/orthanc/plugins/`.
- Auth enabled (`admin/admin` default).
- DICOMweb/OHIF/Explorer2 enabled.
- Python script: `/etc/orthanc/python/startup.py`.
- GDCM and Housekeeper enabled.
- StorageCompression and OverwriteInstances enabled.
