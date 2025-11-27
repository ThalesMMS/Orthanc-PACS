# Quickstart — Orthanc Docker (Linux)

1) Prereqs  
   - Docker + Docker Compose v2 installed (`docker --version`, `docker compose version`).  
   - Make sure Docker is running:  
     - Linux (systemd): `sudo systemctl start docker`  
     - macOS: open Docker Desktop  
   - Ports 8042 (HTTP/REST/UI) and 4242 (DICOM) free.

2) Start  
```bash
docker compose up -d
```

3) URLs  
   - Orthanc Explorer 2 (UI): http://localhost:8042/ui/app/  
   - OHIF viewer: http://localhost:8042/ohif/  
   - DICOMweb: http://localhost:8042/dicom-web/studies  
   - DICOM port: 4242 (AE Title: ORTHANC)  
     - To use host port 104, change to `104:4242` in `docker-compose.yml`, then `docker compose down` + `docker compose up -d`.

4) Credentials (change them)  
   - User: admin  
   - Password: admin  
   - Edit `orthanc-config/orthanc.json` (`RegisteredUsers`) to update.

5) Persistence  
   - `./orthanc-db`: DICOM data + SQLite  
   - `./orthanc-config/orthanc.json`: config  
   - `./orthanc-python/startup.py`: Python plugin script  
   - StorageCompression=true and OverwriteInstances=true already enabled.

6) Quick tweaks  
   - Security: set `RemoteAccessAllowed` to false for production.  
   - AE Title / modalities: edit `DicomAet` and `DicomModalities` in `orthanc.json`.  
   - Reapply after edits: `docker compose up -d`.  
   - To map DICOM to host:104, change port mapping as above.

7) Quick tests  
```bash
curl -u admin:admin http://localhost:8042/system
curl -u admin:admin http://localhost:8042/dicom-web/studies
storescu -aec ORTHANC localhost 4242 *.dcm  # dcmtk send
# If mapped to 104 on host, send to port 104
```

8) Stop / clean  
```bash
docker compose down          # stop, keep data
docker compose down -v       # stop and delete data
```
