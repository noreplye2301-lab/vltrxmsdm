import requests
import json
import time
import threading
import websocket

def scrape(token, guild_id, channel_id):
    """
    Scrapes all members from a Discord guild using the gateway.
    Returns a list of user IDs.
    """
    members = []
    received = threading.Event()
    closed = False

    def on_message(ws, message):
        nonlocal members
        try:
            data = json.loads(message)
        except:
            return

        if data.get('op') == 10:
            hb = data['d']['heartbeat_interval'] / 1000.0

            def heartbeat():
                while not closed:
                    time.sleep(hb)
                    try:
                        ws.send(json.dumps({"op": 1, "d": None}))
                    except:
                        break

            threading.Thread(target=heartbeat, daemon=True).start()

            ws.send(json.dumps({
                "op": 2,
                "d": {
                    "token": token,
                    "properties": {
                        "$os": "Windows",
                        "$browser": "Chrome",
                        "$device": "",
                        "$system_locale": "en-US",
                        "$browser_user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
                        "$browser_version": "120.0.0.0",
                        "$release_channel": "stable",
                        "$client_build_number": 497254
                    }
                }
            }))

        elif data.get('op') == 0:
            t = data.get('t')
            if t == 'READY':
                ws.send(json.dumps({
                    "op": 8,
                    "d": {"guild_id": guild_id, "query": "", "limit": 0}
                }))
            elif t == 'GUILD_MEMBERS_CHUNK':
                chunk = data['d']
                if chunk.get('guild_id') == guild_id:
                    for m in chunk.get('members', []):
                        members.append(m['user']['id'])
                    idx = chunk.get('chunk_index', 0)
                    cnt = chunk.get('chunk_count', 1)
                    print(f"Chunk {idx+1}/{cnt} – total so far: {len(members)}")
                    if idx == cnt - 1:
                        received.set()

        elif data.get('op') == 9:
            print("Invalid session")
            received.set()

    def on_close(ws, code, reason):
        nonlocal closed
        closed = True
        received.set()

    ws = websocket.WebSocketApp(
        "wss://gateway.discord.gg/?v=10&encoding=json",
        on_message=on_message,
        on_close=on_close
    )
    threading.Thread(target=ws.run_forever, daemon=True).start()
    received.wait(timeout=90)
    ws.close()
    return members
