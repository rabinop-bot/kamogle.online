<script lang="ts">
  let matchTimeout: ReturnType<typeof setTimeout> | null = null;
  let matchError = '';
  import ChatInterface from '../lib/components/chat-interface.svelte';
  import ConnectionStatus from '../lib/components/connection-status.svelte';
  import InterestSelector from '../lib/components/interest-selector.svelte';
  import VideoChat from '../lib/components/video-chat.svelte';
import io from 'socket.io-client';
import type { Socket } from 'socket.io-client';

let socket: Socket | null = null;
  let connectionStatus: 'disconnected' | 'connecting' | 'connected' = 'disconnected';
  let messages: { sender: 'me' | 'partner'; text: string; timestamp: number }[] = [];
  let interests: string[] = [];
  let showInterestSelector = true;
  let currentView: 'home' | 'interests' | 'chat' | 'video' = 'home';
  let chatMode: 'text' | 'video' = 'text';

function setupSocket() {
  // Disconnect previous socket if exists
  if (socket) {
    socket.disconnect();
    socket = null;
  }
  connectionStatus = 'connecting';
  matchError = '';
  // Create new socket
  socket = io(window.location.origin, {
    transports: ['websocket'],
    reconnectionAttempts: 3,
    timeout: 5000
  });
  // Logging for debugging
  console.log('Attempting to connect to backend...');
  let chatModeSent = false;
  socket.on('connect', () => {
    if (socket) {
      console.log('Socket connected:', socket.id);
      if (!chatModeSent) {
        socket.emit('chatMode', chatMode);
        chatModeSent = true;
        console.log('Emitted chatMode:', chatMode);
      }
    }
  });
  socket.on('connect_error', (err: unknown) => {
    console.error('Socket connection error:', err);
    matchError = 'Unable to connect to server. Please check your connection.';
    connectionStatus = 'disconnected';
    showInterestSelector = true;
    currentView = 'home';
    if (socket) {
      socket.disconnect();
      socket = null;
    }
  });
  matchTimeout = setTimeout(() => {
    if (connectionStatus !== 'connected') {
      matchError = 'No users available. Please try again later.';
      connectionStatus = 'disconnected';
      showInterestSelector = true;
      currentView = 'home';
      if (socket) {
        socket.disconnect();
        socket = null;
      }
    }
  }, 15000);
  socket.on('matched', () => {
    if (matchTimeout) clearTimeout(matchTimeout);
    connectionStatus = 'connected';
    messages = [];
    showInterestSelector = false;
    currentView = chatMode === 'video' ? 'video' : 'chat';
  });
  socket.on('message', (msg: string) => {
    messages = [...messages, { sender: 'partner', text: msg, timestamp: Date.now() }];
  });
  socket.on('partnerDisconnected', () => {
    connectionStatus = 'disconnected';
    showInterestSelector = true;
    currentView = 'home';
    if (socket) {
      socket.disconnect();
      socket = null;
    }
  });
}

  function handleSendMessage(text: string) {
    messages = [...messages, { sender: 'me', text, timestamp: Date.now() }];
    if (socket !== null && connectionStatus === 'connected') {
      socket.emit('message', text);
    }
  }

  function handleDisconnect() {
    if (matchTimeout) clearTimeout(matchTimeout);
    matchError = '';
    if (socket !== null) {
      socket.disconnect();
      socket = null;
    }
    connectionStatus = 'disconnected';
    showInterestSelector = true;
    currentView = 'home';
  }

  function handleReport() {
    alert('Reported!');
  }

  function handleSaveInterests(selected: string[]) {
    interests = selected;
    showInterestSelector = false;
    // Immediately start chat after interests are saved or skipped
    startChat('text'); // You can change to 'video' if you want video chat by default
  }

  function startChat(mode: 'text' | 'video' = 'text') {
    chatMode = mode;
    setupSocket();
    showInterestSelector = false;
    currentView = mode === 'video' ? 'video' : 'chat';
  }
</script>

<main class="min-h-screen bg-gradient-to-br from-blue-900 via-slate-900 to-cyan-900 py-8 px-2 sm:px-6">
  <div class="max-w-5xl mx-auto">
    <!-- Logo/Header -->
    <header class="text-center mb-6 sm:mb-12">
      <div class="flex items-center justify-center gap-2 sm:gap-4 mb-4 sm:mb-6">
        <div class="relative">
          <img src="https://hebbkx1anhila5yf.public.blob.vercel-storage.com/20250630_155353%20%282%29-xPr04mdMR0Nn9BhHiiQU0F09HIpvfT.png" alt="Kamogle Logo" class="w-12 h-12 sm:w-16 sm:h-16 drop-shadow-2xl animate-pulse" />
          <div class="absolute inset-0 w-12 h-12 sm:w-16 sm:h-16 bg-blue-400/20 rounded-full blur-xl animate-ping"></div>
        </div>
        <h1 class="text-3xl sm:text-4xl md:text-6xl font-bold bg-gradient-to-r from-white via-blue-100 to-cyan-100 bg-clip-text text-transparent drop-shadow-2xl">
          Kamogle
        </h1>
      </div>
      <p class="text-blue-100/80 max-w-3xl mx-auto text-sm sm:text-base md:text-lg leading-relaxed backdrop-blur-sm bg-white/5 rounded-xl sm:rounded-2xl p-4 sm:p-6 border border-white/10">
        Experience the future of anonymous connections. Connect instantly with strangers worldwide through our crystalline-clear platform. 
        <span class="text-cyan-300 font-medium block sm:inline mt-1 sm:mt-0">No registration, pure anonymity, infinite possibilities.</span>
      </p>
    </header>
    <ConnectionStatus status={connectionStatus} />
    {#if matchError}
      <div class="text-center text-red-400 font-semibold mb-4">{matchError}</div>
    {/if}
    {#if showInterestSelector}
      <InterestSelector onSave={handleSaveInterests} currentInterests={interests} />
      <div class="text-center space-y-6 sm:space-y-8 mt-8">
        <div class="flex flex-col sm:grid sm:grid-cols-2 gap-4 sm:gap-6 max-w-2xl mx-auto">
          <button
            onclick={() => startChat('text')}
            class="group relative overflow-hidden bg-gradient-to-r from-blue-600/80 to-cyan-600/80 backdrop-blur-xl border border-blue-400/30 text-white font-semibold py-4 sm:py-6 px-6 sm:px-8 rounded-xl sm:rounded-2xl transition-all duration-500 hover:scale-105 hover:shadow-2xl hover:shadow-blue-500/25 disabled:opacity-50 disabled:cursor-not-allowed active:scale-95"
            disabled={connectionStatus === 'connecting'}
          >
            <div class="absolute inset-0 bg-gradient-to-r from-blue-400/20 to-cyan-400/20 opacity-0 group-hover:opacity-100 transition-opacity duration-500"></div>
            <div class="relative flex items-center justify-center gap-3">
              <svg class="w-5 h-5 sm:w-6 sm:h-6 group-hover:animate-bounce" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
              </svg>
              <span class="text-base sm:text-lg">Text Chat</span>
            </div>
            <div class="absolute inset-0 rounded-xl sm:rounded-2xl bg-gradient-to-r from-transparent via-white/10 to-transparent -skew-x-12 -translate-x-full group-hover:translate-x-full transition-transform duration-1000"></div>
          </button>
          <button
            onclick={() => startChat('video')}
            class="group relative overflow-hidden bg-gradient-to-r from-emerald-600/80 to-teal-600/80 backdrop-blur-xl border border-emerald-400/30 text-white font-semibold py-4 sm:py-6 px-6 sm:px-8 rounded-xl sm:rounded-2xl transition-all duration-500 hover:scale-105 hover:shadow-2xl hover:shadow-emerald-500/25 disabled:opacity-50 disabled:cursor-not-allowed active:scale-95"
            disabled={connectionStatus === 'connecting'}
          >
            <div class="absolute inset-0 bg-gradient-to-r from-emerald-400/20 to-teal-400/20 opacity-0 group-hover:opacity-100 transition-opacity duration-500"></div>
            <div class="relative flex items-center justify-center gap-3">
              <svg class="w-5 h-5 sm:w-6 sm:h-6 group-hover:animate-bounce" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 10l4.553-2.276A1 1 0 0121 8.618v6.764a1 1 0 01-1.447.894L15 14M5 18h8a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z" />
              </svg>
              <span class="text-base sm:text-lg">Video Chat</span>
            </div>
            <div class="absolute inset-0 rounded-xl sm:rounded-2xl bg-gradient-to-r from-transparent via-white/10 to-transparent -skew-x-12 -translate-x-full group-hover:translate-x-full transition-transform duration-1000"></div>
          </button>
        </div>
      </div>
    {:else if currentView === 'chat'}
      <ChatInterface
        messages={messages}
        onSendMessage={handleSendMessage}
        onDisconnect={handleDisconnect}
        onReport={handleReport}
        connectionStatus={connectionStatus}
      />
    {:else if currentView === 'video'}
      <VideoChat
        messages={messages}
        onSendMessage={handleSendMessage}
        onDisconnect={handleDisconnect}
        onReport={handleReport}
        connectionStatus={connectionStatus}
      />
    {/if}
  </div>
</main>
