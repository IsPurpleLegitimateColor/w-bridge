# WhatsApp Bridge (Go)

A Go service that links your local environment to your personal WhatsApp account using the WhatsApp Web multi-device API (via `whatsmeow`). It stores messages in SQLite and exposes a small REST API for sending messages and downloading media. Intended to be used by the MCP server in `../whatsapp-mcp-server`.

## Prerequisites
- Go 1.24+
- Terminal that can display QR codes
- Windows only: CGO + C compiler required for `go-sqlite3`

## Run / Build
- Run (first run prompts a QR code):
  - `cd whatsapp-bridge && go run main.go`
- Build binary:
  - `cd whatsapp-bridge && go build -o whatsapp-bridge`
  - `./whatsapp-bridge`

The service listens on `http://localhost:8080` and prints connection status.

## REST API
- Send text or media
  - `POST /api/send`
  - Body (JSON): `{ "recipient": "<phone or JID>", "message": "hello", "media_path": "/abs/path/to/file.jpg" }`
  - At least one of `message` or `media_path` is required. `recipient` may be a phone number (digits only) or a JID (e.g., `123456789@s.whatsapp.net`, group `...@g.us`).
  - Example (text):
    - `curl -X POST http://localhost:8080/api/send -H 'Content-Type: application/json' -d '{"recipient":"1234567890","message":"Hello"}'`
  - Example (file):
    - `curl -X POST http://localhost:8080/api/send -H 'Content-Type: application/json' -d '{"recipient":"1234567890","media_path":"/path/picture.jpg"}'`
- Download media
  - `POST /api/download`
  - Body (JSON): `{ "message_id": "<id>", "chat_jid": "<jid>" }`
  - Returns JSON with `success`, `filename`, and local `path`.

## Data Storage
- SQLite files under `whatsapp-bridge/store/`:
  - `messages.db` — chats/messages used by the REST API
  - `whatsapp.db` — whatsmeow device/session store

## Windows (CGO)
- Install a C compiler (e.g., MSYS2) and enable CGO:
  - `go env -w CGO_ENABLED=1`
  - Then run/build as above. See root README for details.

## Troubleshooting
- Re-authenticate by re-running and scanning the QR when prompted (periodically required by WhatsApp).
- If state gets out of sync, stop the bridge, delete `store/messages.db` and `store/whatsapp.db`, then start again.
- The API is unauthenticated and bound to localhost; do not expose it publicly.
