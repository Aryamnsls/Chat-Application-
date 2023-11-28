# Chat-Application

Creating a real-time chat application involves several steps, from setting up the project to implementing the server and client-side functionality. Here's a step-by-step guide to creating a real-time chat app using React, Node.js, Socket.io, and MongoDB:

**Step 1: Set Up Your Project**

1. Create a new directory for your project:

```bash
mkdir realtime-chat-app
cd realtime-chat-app
```

2. Initialize a new Node.js project:

```bash
npm init -y
```

**Step 2: Set Up the Server**

1. Install necessary dependencies:

```bash
npm install express socket.io mongoose
```

2. Create a `server.js` file for your Node.js server:

```javascript
// server.js
const express = require('express');
const http = require('http');
const socketIO = require('socket.io');
const mongoose = require('mongoose');

const app = express();
const server = http.createServer(app);
const io = socketIO(server);

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/chatApp', { useNewUrlParser: true, useUnifiedTopology: true });

// Define a simple Message schema
const messageSchema = new mongoose.Schema({
  user: String,
  text: String,
});

const Message = mongoose.model('Message', messageSchema);

// Set up Socket.io connection
io.on('connection', (socket) => {
  console.log('A user connected');

  // Listen for new messages
  socket.on('chat message', async (msg) => {
    const message = new Message(msg);
    await message.save();

    // Broadcast the message to all connected clients
    io.emit('chat message', msg);
  });

  // Disconnect event
  socket.on('disconnect', () => {
    console.log('User disconnected');
  });
});

// Serve static files
app.use(express.static('client/build'));

// Start the server
const PORT = process.env.PORT || 5000;
server.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

**Step 3: Set Up the Client**

1. Create a `client` folder in your project directory.

2. Initialize a new React app inside the `client` folder:

```bash
npx create-react-app .
```

3. Install Socket.io client library:

```bash
npm install socket.io-client
```

4. Update `src/App.js` in the client folder:

```javascript
// src/App.js
import React, { useState, useEffect } from 'react';
import io from 'socket.io-client';

const socket = io('http://localhost:5000');

function App() {
  const [messages, setMessages] = useState([]);
  const [message, setMessage] = useState('');
  const [user, setUser] = useState('');

  useEffect(() => {
    // Listen for incoming messages
    socket.on('chat message', (msg) => {
      setMessages([...messages, msg]);
    });

    return () => {
      // Clean up the socket connection
      socket.disconnect();
    };
  }, [messages]);

  const sendMessage = () => {
    if (user && message) {
      // Emit the message to the server
      socket.emit('chat message', { user, text: message });
      setMessage('');
    }
  };

  return (
    <div>
      <div>
        <input
          type="text"
          placeholder="Enter your username"
          value={user}
          onChange={(e) => setUser(e.target.value)}
        />
      </div>
      <div>
        <ul>
          {messages.map((msg, index) => (
            <li key={index}>{`${msg.user}: ${msg.text}`}</li>
          ))}
        </ul>
      </div>
      <div>
        <input
          type="text"
          placeholder="Type your message"
          value={message}
          onChange={(e) => setMessage(e.target.value)}
        />
        <button onClick={sendMessage}>Send</button>
      </div>
    </div>
  );
}

export default App;
```

**Step 4: Run Your Application**

1. Start the MongoDB server:

```bash
mongod
```

2. Run your Node.js server:

```bash
node server.js
```

3. In a separate terminal, navigate to the `client` folder and start the React app:

```bash
cd client
npm start
```

4. Open your browser and go to `http://localhost:3000` to see the real-time chat app.

That's it! You've successfully created a real-time chat application using React, Node.js, Socket.io, and MongoDB. You can now enhance the app by adding features like user authentication, private messaging, and more.
