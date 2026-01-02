# Azul - Online Multiplayer Board Game

A beautiful web-based implementation of the popular tile-laying board game **Azul**, featuring real-time online multiplayer functionality.

🎮 **[Play Now](https://pooryaem-commits.github.io/azul_game/)**

![Azul Game](https://img.shields.io/badge/Game-Azul-blue)
![Multiplayer](https://img.shields.io/badge/Multiplayer-PeerJS-green)
![Status](https://img.shields.io/badge/Status-Live-brightgreen)

## Features

- 🎨 **Beautiful UI** - Elegant design with smooth animations
- 👥 **Online Multiplayer** - Play with friends in real-time
- 🔗 **Easy Room Codes** - Simple 6-character codes to join games
- 📱 **Responsive** - Works on desktop and mobile browsers
- ⚡ **No Installation** - Runs entirely in the browser
- 🆓 **Free to Play** - No accounts or payments required

## How to Play

### Starting a Game

1. Go to the [game link](https://pooryaem-commits.github.io/azul_game/)
2. Enter your username
3. Click **"Create Room"** to host a game
4. Share the 6-character room code with your friend
5. Wait for them to join - the game starts automatically!

### Joining a Game

1. Go to the [game link](https://pooryaem-commits.github.io/azul_game/)
2. Enter your username
3. Click **"Join Room"**
4. Enter the 6-character room code from your friend
5. The game starts once connected!

## Multiplayer Technology

This game uses **PeerJS** for real-time peer-to-peer multiplayer functionality. This is an excellent solution for adding multiplayer to static HTML games hosted on platforms like GitHub Pages.

### Why PeerJS?

| Feature | Benefit |
|---------|---------|
| **No Backend Required** | Works on static hosting (GitHub Pages, Netlify, etc.) |
| **Free Cloud Server** | PeerJS provides free signaling server |
| **WebRTC Based** | Direct peer-to-peer connection for low latency |
| **Simple API** | Easy to implement in vanilla JavaScript |
| **Reliable** | 99.96% uptime on PeerJS Cloud |

### How It Works

```
┌─────────────┐         ┌─────────────────┐         ┌─────────────┐
│   Player 1  │◄───────►│  PeerJS Cloud   │◄───────►│   Player 2  │
│   (Host)    │         │ (Signaling Only)│         │   (Guest)   │
└─────────────┘         └─────────────────┘         └─────────────┘
       │                                                   │
       │              Direct WebRTC Connection             │
       └───────────────────────────────────────────────────┘
                      (Game Data Transfer)
```

1. **Signaling Phase**: PeerJS Cloud helps players find each other using room codes
2. **Connection Phase**: WebRTC establishes a direct peer-to-peer connection
3. **Game Phase**: All game data flows directly between players (no server)

### Implementation Guide

If you want to add similar multiplayer functionality to your own static HTML game:

#### 1. Include PeerJS

```html
<script src="https://unpkg.com/peerjs@1.5.5/dist/peerjs.min.js"></script>
```

#### 2. Create a Room (Host)

```javascript
const roomCode = generateRoomCode(); // e.g., "ABC123"
const peer = new Peer('game-' + roomCode, {
    config: {
        iceServers: [
            { urls: 'stun:stun.l.google.com:19302' },
            { urls: 'stun:stun1.l.google.com:19302' }
        ]
    }
});

peer.on('open', (id) => {
    console.log('Room created:', roomCode);
});

peer.on('connection', (conn) => {
    // Guest connected!
    conn.on('data', (data) => {
        // Handle incoming game data
    });
});
```

#### 3. Join a Room (Guest)

```javascript
const peer = new Peer(); // Auto-generated ID for guest

peer.on('open', () => {
    const conn = peer.connect('game-' + roomCode);
    
    conn.on('open', () => {
        console.log('Connected to host!');
        conn.send({ type: 'join', username: 'Player2' });
    });
    
    conn.on('data', (data) => {
        // Handle incoming game data
    });
});
```

#### 4. Sync Game State

```javascript
// Host sends game state after each move
function sendGameState() {
    conn.send({
        type: 'gameState',
        state: {
            board: gameBoard,
            currentPlayer: currentPlayer,
            scores: scores
        }
    });
}

// Guest receives and applies state
conn.on('data', (data) => {
    if (data.type === 'gameState') {
        applyGameState(data.state);
    }
});
```

### STUN and TURN Servers

For reliable connections across different networks (NAT traversal), the game uses both STUN and TURN servers:

**STUN servers** help discover your public IP address:
```javascript
{ urls: 'stun:stun.l.google.com:19302' },
{ urls: 'stun:stun1.l.google.com:19302' },
{ urls: 'stun:global.stun.twilio.com:3478' }
```

**TURN servers** relay data when direct connection fails (strict firewalls, symmetric NAT):
```javascript
{
    urls: 'turn:openrelay.metered.ca:80',
    username: 'openrelayproject',
    credential: 'openrelayproject'
},
{
    urls: 'turn:openrelay.metered.ca:443',
    username: 'openrelayproject',
    credential: 'openrelayproject'
},
{
    urls: 'turn:openrelay.metered.ca:443?transport=tcp',
    username: 'openrelayproject',
    credential: 'openrelayproject'
}
```

> **Note**: TURN servers are essential for ~10-15% of connections that can't establish direct peer-to-peer links due to network restrictions.

### Connection Timeout

The game uses a **120-second timeout** for connection attempts. If connection fails:
- Check if room code is correct
- Ensure host hasn't left the room
- Try disabling VPN if using one
- Some corporate/university networks may block WebRTC

### Limitations

- **2 Players**: WebRTC is best for small groups; for larger lobbies, consider a server
- **NAT Issues**: Some strict corporate/university networks may block WebRTC (TURN servers help but don't guarantee 100%)
- **No Persistence**: Game state is lost if both players disconnect
- **Connection Time**: Initial connection may take up to 30 seconds in some network conditions

### Alternatives Considered

| Technology | Pros | Cons |
|------------|------|------|
| **PeerJS** ✅ | Free, simple, no backend | 2-player limit practical |
| Firebase | Real-time, scales well | Requires account, costs $ |
| Ably/PubNub | Professional, reliable | API keys required, costs $ |
| Gun.js | Decentralized, no keys | Unreliable relay servers |
| Custom WebSocket | Full control | Requires server hosting |

## Troubleshooting

### Connection Issues

| Problem | Solution |
|---------|----------|
| "Room not found" | Double-check the 6-character room code |
| Connection timeout | Host should create a new room; both players refresh |
| "Guest connecting..." but never connects | Firewall/NAT issue - try different network or disable VPN |
| Works locally but not remotely | TURN servers should handle this; if not, network is very restrictive |

### Debug Mode

Open browser console (F12 → Console) to see connection logs:
- `Peer created with ID: azul-XXXXXX` - Host room created
- `Guest peer created: xxxx-xxxx-xxxx` - Guest initialized
- `Connecting to host: azul-XXXXXX` - Guest attempting connection
- `Connection opened to host!` - Success! Game should start

### Network Requirements

For best results, ensure:
- Both players have stable internet connection
- WebRTC is not blocked (most networks allow it)
- If using VPN, try disabling it temporarily
- Corporate/school networks may have restrictions

## Game Rules

Azul is a tile-drafting game where players:

1. **Draft Tiles**: Take all tiles of one color from a factory or center
2. **Place Tiles**: Add tiles to pattern lines on your board
3. **Score Points**: Complete rows to move tiles to your wall
4. **Avoid Penalties**: Excess tiles go to the floor line

The game ends when a player completes a horizontal row on their wall. The player with the most points wins!

## Technical Details

- **Frontend**: Vanilla HTML, CSS, JavaScript
- **Hosting**: GitHub Pages
- **Multiplayer**: PeerJS (WebRTC) with STUN/TURN servers
- **Connection Timeout**: 120 seconds
- **No Dependencies**: Single HTML file (~3000 lines)

## Contributing

Feel free to open issues or submit pull requests!

## License

This project is for educational purposes. Azul is a trademark of Plan B Games.

---

Made with ❤️ for board game enthusiasts
