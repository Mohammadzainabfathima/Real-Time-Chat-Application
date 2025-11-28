# Real-Time-Chat-Application
A Java-based real-time chat application featuring instant messaging, live user presence, secure WebSocket communication, and scalable server-client architecture. Built with Java, Spring Boot, and WebSockets to support fast, reliable, and multi-user conversations.
Project structure
spring-realtime-chat/
├── src/
│   ├── main/
│   │   ├── java/com/example/chat/
│   │   │   ├── SpringRealtimeChatApplication.java
│   │   │   ├── config/
│   │   │   │   └── WebSocketConfig.java
│   │   │   ├── controller/
│   │   │   │   └── ChatController.java
│   │   │   └── model/
│   │   │       └── ChatMessage.java
│   │   └── resources/
│   │       ├── static/
│   │       │   ├── index.html
│   │       │   ├── style.css
│   │       │   └── client.js
│   │       └── application.properties
├── .gitignore
├── pom.xml
└── README.md

1) pom.xml

Create this at the repo root:

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
           http://maven.apache.org/POM/4.0.0
           http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>spring-realtime-chat</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>

  <name>spring-realtime-chat</name>

  <properties>
    <java.version>17</java.version>
    <spring.boot.version>3.2.0</spring.boot.version>
  </properties>

  <dependencies>
    <!-- Spring Boot starter web (serves static files) -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>${spring.boot.version}</version>
    </dependency>

    <!-- WebSocket + STOMP support -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-websocket</artifactId>
      <version>${spring.boot.version}</version>
    </dependency>

    <!-- Optional: devtools (remove for production) -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <version>${spring.boot.version}</version>
      <scope>runtime</scope>
      <optional>true</optional>
    </dependency>

    <!-- Testing (optional) -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <version>${spring.boot.version}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>${spring.boot.version}</version>
      </plugin>
    </plugins>
  </build>
</project>


Note: If you use Java 11 or 21, change <java.version> accordingly.

2) SpringRealtimeChatApplication.java

src/main/java/com/example/chat/SpringRealtimeChatApplication.java

package com.example.chat;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringRealtimeChatApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringRealtimeChatApplication.class, args);
    }
}

3) WebSocket configuration — WebSocketConfig.java

src/main/java/com/example/chat/config/WebSocketConfig.java

package com.example.chat.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.*;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // destination prefixes for subscribing (broker)
        config.enableSimpleBroker("/topic");
        // prefix for messages bound for @MessageMapping handlers
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // endpoint clients will use to connect to websocket (with SockJS fallback)
        registry.addEndpoint("/ws-chat").setAllowedOriginPatterns("*").withSockJS();
    }
}

4) Chat controller — ChatController.java

src/main/java/com/example/chat/controller/ChatController.java

package com.example.chat.controller;

import com.example.chat.model.ChatMessage;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

import java.time.Instant;

@Controller
public class ChatController {

    // Client sends to /app/chat.sendMessage
    // This maps to a broker topic /topic/public
    @MessageMapping("/chat.sendMessage")
    @SendTo("/topic/public")
    public ChatMessage sendMessage(@Payload ChatMessage chatMessage) {
        // add server-side timestamp
        chatMessage.setTimestamp(Instant.now().toEpochMilli());
        return chatMessage;
    }

    // Optional: join notifications
    @MessageMapping("/chat.addUser")
    @SendTo("/topic/public")
    public ChatMessage addUser(@Payload ChatMessage chatMessage) {
        chatMessage.setTimestamp(Instant.now().toEpochMilli());
        chatMessage.setType(ChatMessage.MessageType.JOIN);
        return chatMessage;
    }
}

5) Chat message model — ChatMessage.java

src/main/java/com/example/chat/model/ChatMessage.java

package com.example.chat.model;

public class ChatMessage {

    public enum MessageType {
        CHAT,
        JOIN,
        LEAVE
    }

    private MessageType type;
    private String sender;
    private String content;
    private long timestamp;

    public ChatMessage() {
    }

    public MessageType getType() {
        return type;
    }

    public void setType(MessageType type) {
        this.type = type;
    }

    public String getSender() {
        return sender;
    }

    public void setSender(String sender) {
        this.sender = sender;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(long timestamp) {
        this.timestamp = timestamp;
    }

    // convenience factory
    public static ChatMessage chat(String sender, String content) {
        ChatMessage m = new ChatMessage();
        m.setType(MessageType.CHAT);
        m.setSender(sender);
        m.setContent(content);
        return m;
    }
}

6) Frontend — index.html

src/main/resources/static/index.html

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Spring Realtime Chat</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="wrap">
    <aside class="left">
      <h1>Realtime Chat</h1>
      <div class="join">
        <input id="name" type="text" placeholder="Your name" maxlength="30"/>
        <button id="joinBtn">Join</button>
      </div>
      <div id="usersPanel" class="hidden">
        <h3>Room</h3>
        <p>Public (default)</p>
      </div>
    </aside>

    <main class="right">
      <div id="chat" class="chat"></div>

      <form id="msgForm" class="composer hidden">
        <input id="msgInput" autocomplete="off" placeholder="Type a message..." maxlength="1000"/>
        <button type="submit">Send</button>
      </form>
    </main>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/sockjs-client@1/dist/sockjs.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/stompjs@2.3.3/lib/stomp.min.js"></script>
  <script src="client.js"></script>
</body>
</html>

7) Frontend CSS — style.css

src/main/resources/static/style.css

/* small, clean chat UI */
body { margin:0; font-family:Inter,system-ui, -apple-system, "Segoe UI", Roboto, Arial; background:#0b1220; color:#e6eef6; display:flex; align-items:center; justify-content:center; height:100vh; }
.wrap { width:100%; max-width:1000px; height:80vh; background:linear-gradient(180deg,#071022,#061126); border-radius:10px; display:grid; grid-template-columns:260px 1fr; overflow:hidden; box-shadow:0 8px 30px rgba(0,0,0,0.6);}
.left { padding:18px; border-right:1px solid rgba(255,255,255,0.03); }
.left h1 { margin:0 0 12px; font-size:20px; }
.join input { width:100%; padding:10px; border-radius:8px; border:none; margin-bottom:8px; background:rgba(255,255,255,0.02); color:inherit; }
.join button { width:100%; padding:10px; border-radius:8px; border:none; background:#06b6d4; color:#021018; font-weight:600; cursor:pointer; }
.right { display:flex; flex-direction:column; }
.chat { padding:16px; flex:1; overflow:auto; display:flex; flex-direction:column; gap:10px; }
.msg { max-width:70%; background:rgba(255,255,255,0.02); padding:10px 12px; border-radius:10px; }
.msg.me { margin-left:auto; background:rgba(6,182,212,0.12); }
.msg .meta { font-size:12px; color:#94a3b8; margin-bottom:6px; display:flex; gap:8px; align-items:center;}
.composer { display:flex; gap:8px; padding:12px; border-top:1px solid rgba(255,255,255,0.03); }
.composer input { flex:1; padding:10px; border-radius:8px; border:none; background:rgba(255,255,255,0.02); color:inherit; }
.composer button { padding:10px 14px; border-radius:8px; border:none; background:#06b6d4; color:#021018; cursor:pointer; }
.hidden { display:none; }

8) Frontend JS — client.js

src/main/resources/static/client.js

// client.js - connects to /ws-chat, subscribes to /topic/public
let stompClient = null;
const chatEl = document.getElementById('chat');
const joinBtn = document.getElementById('joinBtn');
const nameInput = document.getElementById('name');
const msgForm = document.getElementById('msgForm');
const msgInput = document.getElementById('msgInput');

function connectAndJoin(username) {
  const socket = new SockJS('/ws-chat');
  stompClient = Stomp.over(socket);
  // optional: disable debug logs
  stompClient.debug = null;

  stompClient.connect({}, function(frame) {
    console.log('Connected: ' + frame);
    // notify server we joined (optional)
    const joinMsg = { sender: username, type: 'JOIN', content: username + ' joined' };
    stompClient.send('/app/chat.addUser', {}, JSON.stringify(joinMsg));

    // subscribe to public topic
    stompClient.subscribe('/topic/public', function(message) {
      const payload = JSON.parse(message.body);
      showMessage(payload, username);
    });
  }, function(error) {
    console.error('STOMP error', error);
  });
}

function showMessage(message, myName) {
  const div = document.createElement('div');
  div.className = 'msg' + (message.sender === myName && message.type === 'CHAT' ? ' me' : '');
  const meta = document.createElement('div');
  meta.className = 'meta';
  const sender = document.createElement('span');
  sender.textContent = message.sender || 'System';
  const ts = document.createElement('span');
  const t = message.timestamp ? new Date(message.timestamp) : new Date();
  ts.textContent = t.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
  meta.appendChild(sender);
  meta.appendChild(ts);
  const bubble = document.createElement('div');
  bubble.className = 'bubble';
  if (message.type === 'JOIN') {
    bubble.textContent = `${message.sender} joined the chat`;
    div.classList.add('system');
  } else if (message.type === 'LEAVE') {
    bubble.textContent = `${message.sender} left the chat`;
    div.classList.add('system');
  } else {
    bubble.textContent = message.content;
  }
  div.appendChild(meta);
  div.appendChild(bubble);
  chatEl.appendChild(div);
  chatEl.scrollTop = chatEl.scrollHeight;
}

joinBtn.addEventListener('click', function() {
  const username = nameInput.value.trim() || 'User' + Math.floor(Math.random() * 1000);
  connectAndJoin(username);
  document.querySelector('.join').classList.add('hidden');
  document.querySelector('.composer').classList.remove('hidden');
});

msgForm.addEventListener('submit', function(e) {
  e.preventDefault();
  if (!stompClient) return;
  const msg = msgInput.value.trim();
  if (!msg) return;
  const chatMessage = { sender: nameInput.value || 'Anonymous', content: msg, type: 'CHAT' };
  stompClient.send('/app/chat.sendMessage', {}, JSON.stringify(chatMessage));
  msgInput.value = '';
});

9) application.properties

src/main/resources/application.properties (optional)

# server port (change if needed)
server.port=8080

# logging level (optional)
logging.level.org.springframework.web=INFO

10) .gitignore

At repo root:

# Maven
target/
*.log

# IntelliJ
.idea/
*.iml

# VS Code
.vscode/

# OS
.DS_Store

11) README.md

Polished README for the repo root (copy-paste):

# Spring Realtime Chat

A minimal realtime chat application built with **Spring Boot** and **WebSocket (STOMP)**. The app serves a small static frontend that connects to the server using **SockJS + STOMP**.

## Features
- Real-time messaging using WebSocket + STOMP
- Public topic `/topic/public`
- Simple UI (no build step)
- Single-file model/controller and simple config — easy to extend

## Run locally

Requirements:
- Java 17+ (or change `java.version` in `pom.xml`)
- Maven

Steps:

```bash
git clone https://github.com/your-username/spring-realtime-chat.git
cd spring-realtime-chat
mvn clean package
mvn spring-boot:run
