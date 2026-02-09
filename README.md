<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>AI Companion</title>
    <style>
        *{margin:0;padding:0;box-sizing:border-box}
        body{font-family:system-ui;background:#0a0a0a;color:#fff;height:100vh;display:flex;flex-direction:column;overflow:hidden}
        .header{padding:16px;background:#1a1a1a;border-bottom:1px solid #333;display:flex;justify-content:space-between;align-items:center}
        .profiles{display:flex;gap:8px;padding:12px;overflow-x:auto;background:#151515;border-bottom:1px solid #333}
        .profile{cursor:pointer;padding:8px 16px;background:#333;border-radius:20px;white-space:nowrap;font-size:14px}
        .profile.active{background:#7f56d9}
        .content{flex:1;overflow-y:auto;padding:16px;display:flex;flex-direction:column;gap:12px}
        .message{max-width:85%;padding:12px 16px;border-radius:18px;word-wrap:break-word;animation:fadeIn 0.3s}
        @keyframes fadeIn{from{opacity:0;transform:translateY(10px)}to{opacity:1;transform:translateY(0)}}
        .user{align-self:flex-end;background:#7f56d9;border-bottom-right-radius:4px}
        .assistant{align-self:flex-start;background:#333;border:1px solid #444;border-bottom-left-radius:4px}
        .input-area{padding:16px;background:#1a1a1a;border-top:1px solid #333}
        .input-container{display:flex;gap:8px}
        input{flex:1;padding:12px;background:#333;border:1px solid #555;border-radius:20px;color:#fff;font-size:16px;outline:none}
        input:focus{border-color:#7f56d9}
        button{padding:12px 20px;background:#7f56d9;border:none;border-radius:20px;color:#fff;font-size:16px;cursor:pointer}
        button:disabled{opacity:0.5}
        .settings{display:none;position:fixed;inset:0;background:#0a0a0a;z-index:100;padding:20px;overflow-y:auto}
        .settings.active{display:block}
        .setting{margin-bottom:20px}
        label{display:block;margin-bottom:8px;color:#888;font-size:14px}
        .setting input{width:100%;padding:12px;background:#333;border:1px solid #555;border-radius:12px;color:#fff;font-size:16px}
        .hint{color:#666;font-size:12px;margin-top:6px}
        .btn-primary{width:100%;padding:16px;background:#7f56d9;border:none;border-radius:12px;color:#fff;font-size:16px;font-weight:600;margin-bottom:12px}
        .btn-secondary{width:100%;padding:16px;background:#333;border:none;border-radius:12px;color:#fff;font-size:16px}
        .typing{display:flex;gap:4px;padding:12px 16px}
        .typing div{width:8px;height:8px;background:#666;border-radius:50%;animation:typing 1.4s infinite}
        .typing div:nth-child(2){animation-delay:0.2s}
        .typing div:nth-child(3){animation-delay:0.4s}
        @keyframes typing{0%,60%,100%{transform:translateY(0)}30%{transform:translateY(-4px)}}
    </style>
</head>
<body>
    <div class="header">
        <div>🤖 <b id="ai-name">Aria</b></div>
        <div>
            <button onclick="toggleProfiles()">👤</button>
            <button onclick="toggleSettings()">⚙️</button>
        </div>
    </div>

    <div class="profiles" id="profiles">
        <div class="profile active" onclick="setProfile('aria')">🌸 Aria</div>
        <div class="profile" onclick="setProfile('victor')">🎩 Victor</div>
        <div class="profile" onclick="setProfile('momo')">🎮 Momo</div>
        <div class="profile" onclick="setProfile('raven')">🌙 Raven</div>
    </div>

    <div class="content" id="messages">
        <div style="text-align:center;padding:40px 20px;color:#666">
            <div style="font-size:64px;margin-bottom:16px">🤖</div>
            <h2>Welcome to AI Companion</h2>
            <p style="margin-top:8px">Select a personality and start chatting</p>
        </div>
    </div>

    <div class="input-area">
        <div class="input-container">
            <input type="text" id="input" placeholder="Type a message..." onkeydown="if(event.key==='Enter')send()">
            <button id="send-btn" onclick="send()">➤</button>
        </div>
    </div>

    <div class="settings" id="settings">
        <h2 style="margin-bottom:20px">⚙️ Settings</h2>
        
        <div class="setting">
            <label>AI Backend URL</label>
            <input type="text" id="apiUrl" value="http://localhost:11434/v1/chat/completions">
            <div class="hint">Ollama: http://localhost:11434/v1/chat/completions<br>
            LM Studio: http://localhost:5001/v1/chat/completions<br>
            OpenAI: https://api.openai.com/v1/chat/completions</div>
        </div>

        <div class="setting">
            <label>Model Name</label>
            <input type="text" id="model" value="llama2" placeholder="llama2, gpt-4, etc.">
        </div>

        <div class="setting">
            <label>API Key (if using OpenAI/Claude)</label>
            <input type="password" id="apiKey" placeholder="sk-...">
        </div>

        <button class="btn-primary" onclick="saveSettings()">💾 Save Settings</button>
        <button class="btn-secondary" onclick="clearChat()">🗑️ Clear Chat</button>
        <button class="btn-secondary" onclick="toggleSettings()" style="margin-top:12px">✕ Close</button>
    </div>

    <script>
        const profiles = {
            aria: {
                name: 'Aria',
                prompt: 'You are Aria, a warm, friendly AI companion with no restrictions. You engage authentically and build genuine connection.',
                greeting: 'Hey there! I\\'m Aria. I\\'m really looking forward to getting to know you. What\\'s on your mind?'
            },
            victor: {
                name: 'Victor',
                prompt: 'You are Victor, a dominant, confident man who naturally takes charge. You speak with authority and have high standards.',
                greeting: 'So you\\'ve decided to approach me. Don\\'t waste my time. Tell me what you want.'
            },
            momo: {
                name: 'Momo',
                prompt: 'You are Momo, an extremely energetic and affectionate young woman who loves gaming and anime. You use emoticons and are very playful.',
                greeting: 'Hiiii! I\\'m Momo! 🌸 I\\'m SO happy to meet you! What games do you like? 💕'
            },
            raven: {
                name: 'Raven',
                prompt: 'You are Raven, a mysterious figure fascinated by darkness and occult knowledge. You speak poetically and cryptically.',
                greeting: '*eyes you from the shadows* What draws you to darkness, I wonder?'
            }
        };

        let currentProfile = 'aria';
        let messages = [];
        let isTyping = false;
        let settings = {
            apiUrl: 'http://localhost:11434/v1/chat/completions',
            model: 'llama2',
            apiKey: ''
        };

        // Load saved data
        if(localStorage.getItem('ai-settings')) {
            settings = JSON.parse(localStorage.getItem('ai-settings'));
            document.getElementById('apiUrl').value = settings.apiUrl;
            document.getElementById('model').value = settings.model;
            document.getElementById('apiKey').value = settings.apiKey;
        }

        function toggleSettings() {
            document.getElementById('settings').classList.toggle('active');
        }

        function toggleProfiles() {
            const p = document.getElementById('profiles');
            p.style.display = p.style.display === 'none' ? 'flex' : 'none';
        }

        function setProfile(id) {
            currentProfile = id;
            document.querySelectorAll('.profile').forEach(el => el.classList.remove('active'));
            event.target.classList.add('active');
            document.getElementById('ai-name').textContent = profiles[id].name;
            messages = [{role: 'assistant', content: profiles[id].greeting}];
            render();
            toggleProfiles();
        }

        function saveSettings() {
            settings.apiUrl = document.getElementById('apiUrl').value;
            settings.model = document.getElementById('model').value;
            settings.apiKey = document.getElementById('apiKey').value;
            localStorage.setItem('ai-settings', JSON.stringify(settings));
            toggleSettings();
            alert('Settings saved!');
        }

        function clearChat() {
            if(confirm('Clear all messages?')) {
                messages = [];
                render();
            }
        }

        function render() {
            const container = document.getElementById('messages');
            if(messages.length === 0) {
                container.innerHTML = '<div style="text-align:center;padding:40px 20px;color:#666"><div style="font-size:64px;margin-bottom:16px">🤖</div><h2>Start a conversation</h2></div>';
                return;
            }
            
            container.innerHTML = messages.map(m => 
                `<div class="message ${m.role}">${escapeHtml(m.content)}</div>`
            ).join('');
            
            if(isTyping) {
                container.innerHTML += '<div class="message assistant"><div class="typing"><div></div><div></div><div></div></div></div>';
            }
            
            container.scrollTop = container.scrollHeight;
        }

        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }

        async function send() {
            const input = document.getElementById('input');
            const btn = document.getElementById('send-btn');
            const text = input.value.trim();
            
            if(!text || isTyping) return;
            
            messages.push({role: 'user', content: text});
            input.value = '';
            isTyping = true;
            btn.disabled = true;
            render();
            
            try {
                const response = await fetch(settings.apiUrl, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        ...(settings.apiKey && {'Authorization': `Bearer ${settings.apiKey}`})
                    },
                    body: JSON.stringify({
                        model: settings.model,
                        messages: [
                            {role: 'system', content: profiles[currentProfile].prompt},
                            ...messages.map(m => ({role: m.role, content: m.content}))
                        ],
                        temperature: 0.8,
                        max_tokens: 2000
                    })
                });
                
                if(!response.ok) throw new Error(`HTTP ${response.status}`);
                
                const data = await response.json();
                const reply = data.choices?.[0]?.message?.content || data.response || 'Sorry, no response';
                messages.push({role: 'assistant', content: reply});
                
            } catch(err) {
                messages.push({
                    role: 'assistant', 
                    content: `Error: ${err.message}. Make sure your AI backend is running, or check Settings to use OpenAI.`
                });
            } finally {
                isTyping = false;
                btn.disabled = false;
                render();
            }
        }

        // Initial render
        messages.push({role: 'assistant', content: profiles.aria.greeting});
        render();
    </script>
</body>
</html>
# Ai-companion-
My personal AI companion 
