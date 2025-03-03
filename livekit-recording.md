# LiveKit Server with Egress for Automatic Video Recording

This guide explains how to set up a LiveKit server natively and the Egress service in Docker to enable automatic video recording when a video track is published. It includes token creation, running services, automating recordings with Node.js, and viewing real-time logs.

## Prerequisites

- **Operating System**: Ubuntu (or similar Linux distribution).
- **Docker**: Required for Egress.
- **Node.js**: For automatic recording script (optional).
- **Redis**: For Egress coordination.

### Installation
1. **Install Docker**:
   ```bash
   sudo apt update
   sudo apt install -y docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   docker --version
   ```

2. **Install Redis**:
   ```bash
   sudo apt install -y redis-server
   sudo systemctl start redis
   sudo systemctl enable redis
   redis-cli ping  # Should return "PONG"
   ```

3. **Install Node.js** (Optional, for automation):
   ```bash
   sudo apt install -y nodejs npm
   node -v  # Should be v12+ ideally
   ```

4. **Install livekit-cli**:
   ```bash
   curl -sSL https://get.livekit.io/cli | bash
   livekit-cli --version
   ```

## LiveKit Server Setup (Native)

### Running the Server

1. **Install LiveKit Server**:
   
   Download the binary (replace v1.7.1 with the latest version):
   ```bash
   curl -L -o livekit-server.tar.gz https://github.com/livekit/livekit/releases/download/v1.7.1/livekit-server-linux-amd64.tar.gz
   tar -xzf livekit-server.tar.gz
   sudo mv livekit-server /usr/local/bin/
   livekit-server --version
   ```

2. **Start the Server**:
   
   Run in development mode:
   ```bash
   livekit-server --keys "fhighufdyugifd: ofihgijoeufhdghi" --dev --bind 0.0.0.0
   ```
   Runs on ws://localhost:7880 (unencrypted).

### Checking Real-Time Logs

- **Terminal Output**: Logs are printed to the terminal where the server is running.
- **Redirect to File** (Optional):
  ```bash
  livekit-server --keys "fhighufdyugifd: ofihgijoeufhdghi" --dev --bind 0.0.0.0 > livekit-server.log 2>&1 &
  tail -f livekit-server.log
  ```
  `tail -f`: Shows logs in real time.

## Egress Setup (Docker)

### Running Egress

1. **Prepare Config File**:
   
   Create `/livekit-egress/config.yaml`:
   ```yaml
   api_key: fhighufdyugifd
   api_secret: ofihgijoeufhdghi
   ws_url: ws://localhost:7880
   redis:
     address: localhost:6379
   log_level: debug
   insecure: true
   ```
   
   Ensure the directory exists:
   ```bash
   sudo mkdir -p /livekit-egress
   sudo nano /livekit-egress/config.yaml
   ```

2. **Run Egress in Docker**:
   
   Use a persistent container:
   ```bash
   docker run -d \
     -e EGRESS_CONFIG_FILE=/etc/egress/config.yaml \
     -v /livekit-egress:/etc/egress \
     -v /home/iiii/recordings:/output \
     --cap-add=SYS_ADMIN \
     --network host \
     --name egress \
     livekit/egress
   ```
   - `-d`: Runs in detached mode.
   - `--name egress`: Allows easy management.
   
   Create output directory:
   ```bash
   mkdir -p /home/iiii/recordings
   chmod -R 777 /home/iiii/recordings  # Ensure write access
   ```

### Checking Real-Time Logs

- **View Logs**:
  ```bash
  docker logs -f egress
  ```
  `-f`: Follows logs in real time, showing Egress activity (e.g., Redis/LiveKit connections, recording status).

- **Stop Egress** (If Needed):
  ```bash
  docker stop egress
  ```

## Flow: Token Creation to Automatic Recording

### 1. Token Creation

Generate a JWT for clients to join rooms:

Using livekit-cli:
```bash
livekit-cli create-token \
  --api-key "fhighufdyugifd" \
  --api-secret "ofihgijoeufhdghi" \
  --room "test-room" \
  --identity "user1" \
  --valid-for "24h"
```
Outputs a token (e.g., eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...).

### 2. Client Joins Room

Clients connect using the token:

Using livekit-cli:
```bash
livekit-cli join-room \
  --room "test-room" \
  --url "ws://localhost:7880" \
  --insecure \
  --api-key "fhighufdyugifd" \
  --api-secret "ofihgijoeufhdghi"
```

### 3. Automatic Recording with Node.js

To start recording automatically when a video track is published:

**Create a Node.js Script**:

File: `auto-record.js`
```javascript
const { RoomServiceClient, EgressClient } = require('@livekit/server-sdk');

const roomClient = new RoomServiceClient('ws://localhost:7880', 'fhighufdyugifd', 'ofihgijoeufhdghi', { insecure: true });
const egressClient = new EgressClient('ws://localhost:7880', 'fhighufdyugifd', 'ofihgijoeufhdghi', { insecure: true });

roomClient.on('trackPublished', async (track, participant, room) => {
  if (track.kind === 'video') {
    console.log(`Video track published by ${participant.identity} in ${room.name}`);
    const egressInfo = await egressClient.startRoomCompositeEgress({
      roomName: room.name,
      file: { filepath: `/output/${room.name}-${Date.now()}.mp4` },
    });
    console.log(`Recording started: ${egressInfo.egressId}`);
  }
});

console.log('Listening for video tracks...');
```

Install dependency:
```bash
npm install @livekit/server-sdk
```

Run the Script:
```bash
node auto-record.js
```
Keeps running, triggering Egress when a video starts.

### 4. Video Recording

- **Egress Behavior**: Once started (via Docker), Egress remains active and handles recording requests. You don't need to restart it for each recording—just trigger new jobs (manual or automatic).
- **Output**: Recordings save to `/home/iiii/recordings/` (mapped to `/output` in Docker).

### 5. Stopping Recordings

**Manual Stop**:
```bash
livekit-cli egress stop --egress-id <id>
```
Get `<id>` from the Node.js script output or livekit-cli start command.

## Testing the Setup

1. **Start LiveKit Server**:
   ```bash
   livekit-server --keys "fhighufdyugifd: ofihgijoeufhdghi" --dev --bind 0.0.0.0
   ```

2. **Start Egress**:
   ```bash
   docker run -d \
     -e EGRESS_CONFIG_FILE=/etc/egress/config.yaml \
     -v /livekit-egress:/etc/egress \
     -v /home/iiii/recordings:/output \
     --cap-add=SYS_ADMIN \
     --network host \
     --name egress \
     livekit/egress
   ```

3. **Run Auto-Recording Script**:
   ```bash
   node auto-record.js
   ```

4. **Join Room and Publish Video**:
   ```bash
   livekit-cli join-room \
     --room "test-room" \
     --url "ws://localhost:7880" \
     --insecure \
     --api-key "fhighufdyugifd" \
     --api-secret "ofihgijoeufhdghi"
   ```
   Note: CLI might not publish video by default. Use a LiveKit client (e.g., Web Quickstart) for testing.

5. **Check Logs**:
   - LiveKit Server: `tail -f livekit-server.log` (if redirected) or terminal.
   - Egress: `docker logs -f egress`.

6. **Verify Recording**:
   ```bash
   ls /home/iiii/recordings/
   ```

## Notes

- **Egress Persistence**: No need to restart Egress for each recording—it runs continuously until the container stops.
- **TLS**: This setup uses `--insecure` (no TLS). For production, configure TLS with certificates and use `wss://`.
- **Troubleshooting**: If recordings don't start, check Egress logs (`docker logs -f egress`) for errors (e.g., Redis connection, file permissions).
