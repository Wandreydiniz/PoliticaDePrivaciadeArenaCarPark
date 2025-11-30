<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>DayZombi - Login</title>

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <style>
        body {
            background: #0f0f0f;
            font-family: Arial;
            color: white;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
        }

        .container {
            width: 100%;
            max-width: 420px;
            padding: 20px;
        }

        /* CARD LIMPO */
        .card {
            background: #181818;
            padding: 18px;
            border-radius: 12px;
            border: 1px solid #222;
            margin-bottom: 20px;
        }

        h1, h2, h3 {
            font-weight: normal;
            margin: 0 0 10px 0;
            text-align: center;
        }

        h1 { color: #f0f0f0; font-size: 22px; }
        h2 { color: #dedede; font-size: 18px; }
        h3 { color: #b8b8b8; font-size: 15px; }

        /* INPUT PROFISSIONAL */
        input {
            width: 100%;
            padding: 12px;
            margin-top: 10px;
            background: #111;
            border: 1px solid #333;
            border-radius: 8px;
            color: white;
            font-size: 15px;
        }

        /* BOTÃO CLEAN */
        button {
            width: 100%;
            padding: 12px;
            background: #2e2e2e;
            border: 1px solid #444;
            border-radius: 8px;
            font-size: 15px;
            color: white;
            margin-top: 12px;
            cursor: pointer;
        }

        button:active {
            transform: scale(0.98);
        }

        .logout {
            background: #2b2b2b;
            border-color: #555;
        }

        /* CHAT PROFISSIONAL */
        #chatSection {
            display: none;
        }

        .chat-box {
            height: 260px;
            overflow-y: auto;
            background: #121212;
            border-radius: 10px;
            padding: 12px;
            border: 1px solid #222;
        }

        .msg {
            background: #1e1e1e;
            padding: 10px;
            border-radius: 8px;
            margin-bottom: 6px;
            font-size: 14px;
        }

        .msg strong {
            color: #c7c7c7;
        }

        .chat-input {
            display: flex;
            margin-top: 12px;
            gap: 8px;
        }

        .chat-input input {
            flex: 1;
            padding: 12px;
        }

    </style>

    <!-- ===== FIREBASE ===== -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { 
            getAuth, createUserWithEmailAndPassword, 
            signInWithEmailAndPassword, signOut, onAuthStateChanged 
        } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js";

        import { 
            getDatabase, ref, push, onValue 
        } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

        const firebaseConfig = {
            apiKey: "AIzaSyB5A-ySceXCFRQ7iSCnOA68nRJqYpK6DQc",
            authDomain: "dayzozmbi-server.firebaseapp.com",
            databaseURL: "https://dayzozmbi-server-default-rtdb.firebaseio.com",
            projectId: "dayzozmbi-server",
            storageBucket: "dayzozmbi-server.firebasestorage.app"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getDatabase(app);

        const loginCard = document.getElementById("loginCard");
        const profileCard = document.getElementById("profileCard");
        const chatBox = document.getElementById("chatBox");

        let playerName = "";

        // AUTO LOGIN
        onAuthStateChanged(auth, (user) => {
            if (user) {
                loginCard.style.display = "none";
                profileCard.style.display = "block";
                document.getElementById("profileEmail").value = user.email;
            } else {
                loginCard.style.display = "block";
                profileCard.style.display = "none";
            }
        });

        // LOGIN
        window.Login = () => {
            signInWithEmailAndPassword(auth,
                document.getElementById("email").value,
                document.getElementById("senha").value
            ).catch(e => alert(e.message));
        };

        // CADASTRAR
        window.Cadastrar = () => {
            createUserWithEmailAndPassword(auth,
                document.getElementById("email").value,
                document.getElementById("senha").value
            ).catch(e => alert(e.message));
        };

        // LOGOUT
        window.Logout = () => signOut(auth);

        // ENTRAR NO CHAT
        window.EntrarChat = () => {
            playerName = document.getElementById("profileName").value.trim();
            if (playerName.length < 2) return alert("Nome muito curto.");
            document.getElementById("chatSection").style.display = "block";
        };

        // ENVIAR MENSAGEM
        window.EnviarMensagem = () => {
            let msg = document.getElementById("chatText").value.trim();
            if (!msg) return;
            push(ref(db, "globalChat"), { nome: playerName, msg });
            document.getElementById("chatText").value = "";
        };

        // RECEBER MENSAGENS
        onValue(ref(db, "globalChat"), (snap) => {
            chatBox.innerHTML = "";
            snap.forEach(c => {
                let d = c.val();
                let div = document.createElement("div");
                div.className = "msg";
                div.innerHTML = `<strong>${d.nome}:</strong> ${d.msg}`;
                chatBox.appendChild(div);
            });
            chatBox.scrollTop = chatBox.scrollHeight;
        });

    </script>

</head>
<body>

    <div class="container">

        <!-- LOGIN -->
        <div class="card" id="loginCard">
            <h1>DayZombi</h1>
            <h3>Entrar</h3>

            <input id="email" type="email" placeholder="E-mail">
            <input id="senha" type="password" placeholder="Senha">

            <button onclick="Login()">Entrar</button>
            <button onclick="Cadastrar()">Criar Conta</button>
        </div>

        <!-- PERFIL + CHAT -->
        <div class="card" id="profileCard" style="display:none;">
            <h2>Perfil</h2>

            <input id="profileName" type="text" placeholder="Seu nome no chat">
            <input id="profileEmail" type="email" disabled>

            <button onclick="EntrarChat()">Entrar no Chat</button>

            <!-- CHAT -->
            <div id="chatSection">
                <h3>Chat Global</h3>

                <div id="chatBox" class="chat-box"></div>

                <div class="chat-input">
                    <input id="chatText" type="text" placeholder="Mensagem...">
                    <button onclick="EnviarMensagem()">OK</button>
                </div>
            </div>

            <button class="logout" onclick="Logout()">Sair</button>
        </div>

    </div>

</body>
</html>
*aaa
