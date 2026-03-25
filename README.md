<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SK AI Chat</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        /* Custom scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #888; border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: #555; }

        /* Mobile specific heights to prevent keyboard overlap */
        :root { --vh: 1vh; }
        body { min-height: 100vh; min-height: -webkit-fill-available; }
        .app-container { height: 100dvh; }
        
        /* Markdown Styling */
        .prose pre { background: #1e1e1e; color: #d4d4d4; padding: 1rem; border-radius: 0.5rem; overflow-x: auto; margin: 0.5rem 0; }
        .prose code { font-family: 'Fira Code', monospace; font-size: 0.9em; }
        .dark .prose { color: #e5e7eb; }

        /* Animations */
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .message-fade { animation: fadeIn 0.3s ease forwards; }
        
        #loading-overlay {
            position: fixed; inset: 0; background: white; z-index: 100;
            display: flex; align-items: center; justify-content: center;
        }
        .dark #loading-overlay { background: #111827; color: white; }
    </style>
</head>
<body class="bg-gray-50 dark:bg-gray-900 transition-colors duration-200">

    <div id="loading-overlay">
        <div class="flex flex-col items-center">
            <i class="fas fa-circle-notch fa-spin text-3xl text-blue-600 mb-4"></i>
            <p class="font-medium">Initializing SK AI...</p>
        </div>
    </div>

    <div id="app" class="app-container flex overflow-hidden">
        <aside id="sidebar" class="fixed inset-y-0 left-0 z-40 w-72 bg-gray-100 dark:bg-gray-800 transform -translate-x-full lg:translate-x-0 transition-transform duration-300 ease-in-out border-r dark:border-gray-700 flex flex-col">
            <div class="p-4 border-b dark:border-gray-700 flex justify-between items-center">
                <h1 class="font-bold text-xl text-blue-600 dark:text-blue-400">SK AI Chat</h1>
                <button onclick="toggleSidebar()" class="lg:hidden p-2 text-gray-500"><i class="fas fa-times"></i></button>
            </div>
            
            <button onclick="createNewChat()" class="m-4 p-3 bg-white dark:bg-gray-700 border dark:border-gray-600 rounded-xl hover:bg-gray-50 dark:hover:bg-gray-600 flex items-center gap-3 transition shadow-sm">
                <i class="fas fa-plus text-blue-500"></i>
                <span class="font-medium">New Chat</span>
            </button>

            <div id="chat-list" class="flex-1 overflow-y-auto px-2 space-y-1">
                </div>

            <div class="p-4 border-t dark:border-gray-700 space-y-2">
                <button onclick="toggleDarkMode()" class="w-full flex items-center gap-3 p-2 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg transition">
                    <i class="fas fa-moon dark:hidden"></i><i class="fas fa-sun hidden dark:block"></i>
                    <span>Appearance</span>
                </button>
                <button onclick="openSettings()" class="w-full flex items-center gap-3 p-2 hover:bg-gray-200 dark:hover:bg-gray-700 rounded-lg transition">
                    <i class="fas fa-cog"></i>
                    <span>Settings</span>
                </button>
            </div>
        </aside>

        <main class="flex-1 flex flex-col min-w-0 bg-white dark:bg-gray-900 relative">
            <header class="h-16 border-b dark:border-gray-800 flex items-center px-4 justify-between bg-white/80 dark:bg-gray-900/80 backdrop-blur-md sticky top-0 z-30">
                <div class="flex items-center gap-3">
                    <button onclick="toggleSidebar()" class="lg:hidden p-2 text-gray-500"><i class="fas fa-bars text-xl"></i></button>
                    <h2 id="current-chat-title" class="font-semibold truncate max-w-[200px]">New Conversation</h2>
                </div>
                <div id="model-badge" class="text-xs bg-blue-100 text-blue-700 px-2 py-1 rounded-full dark:bg-blue-900/30 dark:text-blue-300">
                    deepseek/deepseek-r1:free
                </div>
            </header>

            <div id="messages-container" class="flex-1 overflow-y-auto p-4 md:p-8 space-y-6">
                <div id="empty-state" class="h-full flex flex-col items-center justify-center text-center space-y-4 opacity-50">
                    <div class="w-16 h-16 bg-blue-100 dark:bg-gray-800 rounded-full flex items-center justify-center">
                        <i class="fas fa-robot text-2xl text-blue-500"></i>
                    </div>
                    <h3 class="text-xl font-medium">How can I help you today?</h3>
                </div>
            </div>

            <div class="p-4 bg-gradient-to-t from-white via-white dark:from-gray-900 dark:via-gray-900 to-transparent">
                <div class="max-w-4xl mx-auto relative group">
                    <textarea 
                        id="user-input"
                        rows="1"
                        placeholder="Message SK AI..."
                        class="w-full p-4 pr-14 bg-gray-100 dark:bg-gray-800 border-none rounded-2xl focus:ring-2 focus:ring-blue-500 focus:bg-white dark:focus:bg-gray-700 transition-all resize-none shadow-sm dark:text-white"
                        onkeydown="handleKeydown(event)"
                    ></textarea>
                    <button 
                        id="send-btn"
                        onclick="sendMessage()"
                        class="absolute right-3 bottom-3 p-2 px-3 bg-blue-600 text-white rounded-xl hover:bg-blue-700 disabled:opacity-30 disabled:cursor-not-allowed transition-all"
                    >
                        <i class="fas fa-paper-plane"></i>
                    </button>
                </div>
                <p class="text-center text-[10px] mt-2 text-gray-400">SK AI can make mistakes. Check important info.</p>
            </div>
        </main>
    </div>

    <div id="settings-modal" class="fixed inset-0 z-50 hidden bg-black/50 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="bg-white dark:bg-gray-800 rounded-2xl w-full max-w-md shadow-2xl">
            <div class="p-6 border-b dark:border-gray-700 flex justify-between items-center">
                <h3 class="text-xl font-bold">Settings</h3>
                <button onclick="closeSettings()"><i class="fas fa-times"></i></button>
            </div>
            <div class="p-6 space-y-4">
                <div>
                    <label class="block text-sm font-medium mb-1">OpenRouter API Key</label>
                    <input type="password" id="api-key-input" class="w-full p-2 border dark:border-gray-600 rounded-lg dark:bg-gray-700" placeholder="sk-or-v1-...">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Model ID</label>
                    <input type="text" id="model-input" class="w-full p-2 border dark:border-gray-600 rounded-lg dark:bg-gray-700" value="google/gemini-2.0-flash-exp:free">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">System Prompt</label>
                    <textarea id="system-prompt-input" rows="3" class="w-full p-2 border dark:border-gray-600 rounded-lg dark:bg-gray-700" placeholder="You are a helpful assistant..."></textarea>
                </div>
                <button onclick="saveSettings()" class="w-full bg-blue-600 text-white py-3 rounded-xl font-bold hover:bg-blue-700 transition">Save Changes</button>
            </div>
        </div>
    </div>

    <script>
        // --- State Management ---
        let state = {
            apiKey: localStorage.getItem('sk_api_key') || '',
            model: localStorage.getItem('sk_model') || 'google/gemini-2.0-flash-exp:free',
            systemPrompt: localStorage.getItem('sk_system_prompt') || 'You are a helpful assistant.',
            chats: JSON.parse(localStorage.getItem('sk_chats')) || [],
            currentChatId: null,
            isDark: localStorage.getItem('sk_dark') === 'true',
            isStreaming: false
        };

        const elements = {
            chatList: document.getElementById('chat-list'),
            msgContainer: document.getElementById('messages-container'),
            userInput: document.getElementById('user-input'),
            sendBtn: document.getElementById('send-btn'),
            settingsModal: document.getElementById('settings-modal'),
            currentTitle: document.getElementById('current-chat-title'),
            emptyState: document.getElementById('empty-state'),
            loader: document.getElementById('loading-overlay')
        };

        // --- Initialization ---
        window.addEventListener('DOMContentLoaded', () => {
            initApp();
            // Hide loader with a slight delay for smooth feel
            setTimeout(() => elements.loader.style.display = 'none', 500);
        });

        function initApp() {
            if (state.isDark) document.documentElement.classList.add('dark');
            renderChatList();
            if (state.chats.length > 0) {
                loadChat(state.chats[0].id);
            } else {
                createNewChat();
            }
            
            // Auto-resize textarea
            elements.userInput.addEventListener('input', function() {
                this.style.height = 'auto';
                this.style.height = (this.scrollHeight) + 'px';
                if(this.scrollHeight > 200) this.style.overflowY = 'scroll';
            });
        }

        // --- Core Functions ---
        async function sendMessage() {
            const text = elements.userInput.value.trim();
            if (!text || !state.apiKey || state.isStreaming) {
                if(!state.apiKey) alert('Please enter your API Key in Settings');
                return;
            }

            const currentChat = state.chats.find(c => c.id === state.currentChatId);
            const userMsg = { role: 'user', content: text };
            currentChat.messages.push(userMsg);
            
            // UI Updates
            elements.userInput.value = '';
            elements.userInput.style.height = 'auto';
            renderMessages();
            saveState();

            // Prepare AI message placeholder
            state.isStreaming = true;
            elements.sendBtn.disabled = true;
            const aiMsg = { role: 'assistant', content: '' };
            currentChat.messages.push(aiMsg);
            const msgIdx = currentChat.messages.length - 1;
            
            const aiMsgEl = createMessageElement('assistant', '');
            elements.msgContainer.appendChild(aiMsgEl);
            const contentEl = aiMsgEl.querySelector('.msg-content');
            
            elements.msgContainer.scrollTo(0, elements.msgContainer.scrollHeight);

            try {
                const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${state.apiKey}`,
                        'Content-Type': 'application/json',
                        'HTTP-Referer': window.location.href,
                        'X-Title': 'SK AI Clone'
                    },
                    body: JSON.stringify({
                        model: state.model,
                        messages: [
                            { role: 'system', content: state.systemPrompt },
                            ...currentChat.messages.slice(0, -1)
                        ],
                        stream: true
                    })
                });

                const reader = response.body.getReader();
                const decoder = new TextDecoder();
                let fullContent = '';

                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    
                    const chunk = decoder.decode(value);
                    const lines = chunk.split('\n').filter(line => line.trim() !== '');
                    
                    for (const line of lines) {
                        const message = line.replace(/^data: /, '');
                        if (message === '[DONE]') break;
                        try {
                            const parsed = JSON.parse(message);
                            const delta = parsed.choices[0].delta?.content || '';
                            fullContent += delta;
                            currentChat.messages[msgIdx].content = fullContent;
                            contentEl.innerHTML = marked.parse(fullContent);
                            elements.msgContainer.scrollTo(0, elements.msgContainer.scrollHeight);
                        } catch (e) { /* Ignore partial JSON */ }
                    }
                }
            } catch (error) {
                console.error(error);
                contentEl.innerHTML = `<span class="text-red-500">Error: ${error.message}</span>`;
            } finally {
                state.isStreaming = false;
                elements.sendBtn.disabled = false;
                saveState();
                renderChatList();
            }
        }

        // --- UI Rendering ---
        function renderMessages() {
            const currentChat = state.chats.find(c => c.id === state.currentChatId);
            elements.msgContainer.innerHTML = '';
            
            if (!currentChat || currentChat.messages.length === 0) {
                elements.emptyState.style.display = 'flex';
                return;
            }
            
            elements.emptyState.style.display = 'none';
            currentChat.messages.forEach(msg => {
                elements.msgContainer.appendChild(createMessageElement(msg.role, msg.content));
            });
            elements.msgContainer.scrollTo(0, elements.msgContainer.scrollHeight);
        }

        function createMessageElement(role, content) {
            const div = document.createElement('div');
            div.className = `flex ${role === 'user' ? 'justify-end' : 'justify-start'} message-fade`;
            
            const inner = `
                <div class="max-w-[85%] md:max-w-[70%] rounded-2xl p-4 ${
                    role === 'user' 
                    ? 'bg-blue-600 text-white rounded-tr-none' 
                    : 'bg-gray-100 dark:bg-gray-800 text-gray-800 dark:text-gray-100 rounded-tl-none'
                }">
                    <div class="text-[10px] uppercase font-bold mb-1 opacity-50">${role}</div>
                    <div class="prose prose-sm max-w-none msg-content">${marked.parse(content || '...')}</div>
                </div>
            `;
            div.innerHTML = inner;
            return div;
        }

        function renderChatList() {
            elements.chatList.innerHTML = '';
            state.chats.forEach(chat => {
                const btn = document.createElement('div');
                btn.className = `group flex items-center justify-between p-3 rounded-lg cursor-pointer transition ${state.currentChatId === chat.id ? 'bg-blue-50 dark:bg-blue-900/20 text-blue-600' : 'hover:bg-gray-200 dark:hover:bg-gray-700'}`;
                btn.onclick = () => loadChat(chat.id);
                
                btn.innerHTML = `
                    <div class="flex items-center gap-3 truncate">
                        <i class="far fa-comment-alt"></i>
                        <span class="truncate text-sm font-medium">${chat.title}</span>
                    </div>
                    <button onclick="deleteChat(event, '${chat.id}')" class="opacity-0 group-hover:opacity-100 p-1 hover:text-red-500 transition">
                        <i class="fas fa-trash-alt text-xs"></i>
                    </button>
                `;
                elements.chatList.appendChild(btn);
            });
        }

        // --- Helper Functions ---
        function createNewChat() {
            const id = Date.now().toString();
            state.chats.unshift({
                id,
                title: 'New Chat',
                messages: [],
                created: new Date().toISOString()
            });
            state.currentChatId = id;
            saveState();
            renderChatList();
            renderMessages();
            elements.currentTitle.innerText = 'New Conversation';
            if(window.innerWidth < 1024) toggleSidebar();
        }

        function loadChat(id) {
            state.currentChatId = id;
            const chat = state.chats.find(c => c.id === id);
            elements.currentTitle.innerText = chat.title;
            renderMessages();
            renderChatList();
            if(window.innerWidth < 1024) toggleSidebar();
        }

        function deleteChat(e, id) {
            e.stopPropagation();
            state.chats = state.chats.filter(c => c.id !== id);
            if (state.currentChatId === id) {
                state.currentChatId = state.chats.length > 0 ? state.chats[0].id : null;
            }
            if (!state.currentChatId) createNewChat();
            saveState();
            renderChatList();
            renderMessages();
        }

        function saveState() {
            localStorage.setItem('sk_chats', JSON.stringify(state.chats));
            localStorage.setItem('sk_api_key', state.apiKey);
            localStorage.setItem('sk_model', state.model);
            localStorage.setItem('sk_system_prompt', state.systemPrompt);
            localStorage.setItem('sk_dark', state.isDark);
        }

        function toggleSidebar() {
            const sidebar = document.getElementById('sidebar');
            sidebar.classList.toggle('-translate-x-full');
        }

        function toggleDarkMode() {
            state.isDark = !state.isDark;
            document.documentElement.classList.toggle('dark');
            saveState();
        }

        function openSettings() {
            document.getElementById('api-key-input').value = state.apiKey;
            document.getElementById('model-input').value = state.model;
            document.getElementById('system-prompt-input').value = state.systemPrompt;
            elements.settingsModal.classList.remove('hidden');
        }

        function closeSettings() {
            elements.settingsModal.classList.add('hidden');
        }

        function saveSettings() {
            state.apiKey = document.getElementById('api-key-input').value;
            state.model = document.getElementById('model-input').value;
            state.systemPrompt = document.getElementById('system-prompt-input').value;
            document.getElementById('model-badge').innerText = state.model;
            saveState();
            closeSettings();
        }

        function handleKeydown(e) {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                sendMessage();
            }
        }

        // Global Error Handling
        window.onerror = function(msg, url, line) {
            console.error("Critical Error: " + msg + " at " + line);
            return false;
        };
    </script>
</body>
</html>
