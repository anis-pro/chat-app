// استيراد المكتبات
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

// إعداد التطبيق
const app = express();
const server = http.createServer(app);
const io = new Server(server);

// تقديم واجهة HTML
app.get("/", (req, res) => {
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Chat App</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: center;
                height: 100vh;
                margin: 0;
            }
            #messages {
                border: 1px solid #ddd;
                width: 80%;
                height: 50vh;
                overflow-y: scroll;
                padding: 10px;
                margin-bottom: 20px;
            }
            #form {
                display: flex;
                width: 80%;
            }
            #input {
                flex: 1;
                padding: 10px;
                border: 1px solid #ddd;
            }
            #send {
                padding: 10px 20px;
                border: none;
                background-color: #28a745;
                color: white;
                cursor: pointer;
            }
            #send:hover {
                background-color: #218838;
            }
        </style>
    </head>
    <body>
        <h1>Chat Room</h1>
        <div id="messages"></div>
        <form id="form">
            <input id="input" autocomplete="off" placeholder="Type a message..." />
            <button id="send" type="submit">Send</button>
        </form>

        <script src="/socket.io/socket.io.js"></script>
        <script>
            const socket = io();

            // عناصر الواجهة
            const form = document.getElementById("form");
            const input = document.getElementById("input");
            const messages = document.getElementById("messages");

            // إرسال الرسالة
            form.addEventListener("submit", (e) => {
                e.preventDefault();
                if (input.value) {
                    socket.emit("chat message", input.value);
                    input.value = "";
                }
            });

            // استقبال الرسائل
            socket.on("chat message", (msg) => {
                const item = document.createElement("div");
                item.textContent = msg;
                messages.appendChild(item);
                messages.scrollTop = messages.scrollHeight; // التمرير التلقائي
            });
        </script>
    </body>
    </html>
    `);
});

// الاستماع على الاتصالات
io.on("connection", (socket) => {
    console.log("User connected:", socket.id);

    // استقبال الرسائل من المستخدمين وبثها للجميع
    socket.on("chat message", (msg) => {
        io.emit("chat message", msg);
    });

    // التعامل مع انقطاع الاتصال
    socket.on("disconnect", () => {
        console.log("User disconnected:", socket.id);
    });
});

// تشغيل الخادم
const PORT = 3000;
server.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
