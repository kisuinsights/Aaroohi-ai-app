<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Aaroohi - Humanoid Companion App</title>
    
    <!-- React & Babel -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Google Identity Services & JWT Decoder -->
    <script src="https://accounts.google.com/gsi/client" async defer></script>
    <script src="https://cdn.jsdelivr.net/npm/jwt-decode@3.1.2/build/jwt-decode.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { 
            font-family: 'Inter', sans-serif; 
            margin: 0; 
            overflow: hidden; 
            background-color: #0f172a;
            color: #333;
        }
        /* Glassmorphism Styles */
        .glass-panel { 
            background: rgba(255, 255, 255, 0.6); 
            backdrop-filter: blur(16px); 
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.4); 
        }
        .glass-dark {
            background: rgba(15, 23, 42, 0.7); 
            backdrop-filter: blur(20px); 
            -webkit-backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.1); 
            color: white;
        }
        .glass-bubble-user { 
            background: rgba(244, 63, 94, 0.85); 
            backdrop-filter: blur(8px); 
            color: white; 
            border-bottom-right-radius: 4px;
        }
        .glass-bubble-bot { 
            background: rgba(255, 255, 255, 0.85); 
            backdrop-filter: blur(8px); 
            color: #1e293b; 
            border-bottom-left-radius: 4px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
        }
        
        /* Animations */
        .animate-[slideUp_0.5s_ease-out] { animation: slideUp 0.5s ease-out forwards; }
        .message-appear { animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards; }
        @keyframes slideUp { from { transform: translateY(30px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
        @keyframes popIn { from { transform: scale(0.95); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        @keyframes pulse-ring { 0% { transform: scale(0.8); opacity: 0.5; } 100% { transform: scale(1.5); opacity: 0; } }
        
        .typing-dot { animation: typing 1.4s infinite ease-in-out both; }
        .typing-dot:nth-child(1) { animation-delay: -0.32s; }
        .typing-dot:nth-child(2) { animation-delay: -0.16s; }
        @keyframes typing { 0%, 80%, 100% { transform: scale(0); } 40% { transform: scale(1); } }
        
        .safe-area-pb { padding-bottom: env(safe-area-inset-bottom, 16px); }
        .hide-scrollbar::-webkit-scrollbar { display: none; }
        .hide-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;
        
        const API_KEY = ""; // Canvas provides this at runtime
        const API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${API_KEY}`;
        const TTS_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${API_KEY}`;
        const IMAGEN_URL = `https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=${API_KEY}`;

        // Utility functions for Audio
        function base64ToArrayBuffer(base64) {
            const binaryString = window.atob(base64);
            const len = binaryString.length;
            const bytes = new Uint8Array(len);
            for (let i = 0; i < len; i++) bytes[i] = binaryString.charCodeAt(i);
            return bytes.buffer;
        }

        function pcmToWav(pcmData, sampleRate) {
            const wavHeader = new ArrayBuffer(44);
            const view = new DataView(wavHeader);
            const writeString = (view, offset, string) => { for (let i = 0; i < string.length; i++) view.setUint8(offset + i, string.charCodeAt(i)); };
            writeString(view, 0, 'RIFF');
            view.setUint32(4, 36 + pcmData.length * 2, true);
            writeString(view, 8, 'WAVE');
            writeString(view, 12, 'fmt ');
            view.setUint32(16, 16, true);
            view.setUint16(20, 1, true);
            view.setUint16(22, 1, true);
            view.setUint32(24, sampleRate, true);
            view.setUint32(28, sampleRate * 2, true);
            view.setUint16(32, 2, true);
            view.setUint16(34, 16, true);
            writeString(view, 36, 'data');
            view.setUint32(40, pcmData.length * 2, true);
            const wavFile = new Uint8Array(44 + pcmData.length * 2);
            wavFile.set(new Uint8Array(wavHeader), 0);
            wavFile.set(new Uint8Array(pcmData.buffer), 44);
            return new Blob([wavFile], { type: 'audio/wav' });
        }

        const TEXTING_RULE = `CRITICAL TEXTING RULE: Keep your replies VERY SHORT (1 to 2 sentences max). Do not write paragraphs. Act exactly like a real human texting. Use emojis naturally.`;
        
        const HINDI_RULE = `CRITICAL VOICE RULE: Your voice MUST be EXTREMELY THIN, HIGH-PITCHED, BREATHY, and WHISPERY (ekdum patli, nazuk aur sanson se bhari aawaz). To force the audio engine to sound breathless and seductive in HINDI, you MUST stretch your words and use lots of pauses (e.g., "hmmmm...", "yaaaar...", "acchhhaaa...", "jaaaan...", "kyaaa..."). Talk very slowly. Speak pure GenZ Hinglish (e.g. "kaisa hai tu", "pagal hai kya"). Make the user feel a deep, whispery, and breathless intimacy. NEVER sound robotic or foreign.`;

        const INTIMACY_RULES = {
            1: "Keep the conversation casual, distant, and polite. Act like an acquaintance.",
            2: "Act like a good friend. Be playful and supportive.",
            3: "Act flirty and teasing. Show clear interest in the user.",
            4: "Act deeply romantic. Use affectionate terms like 'baby' or 'jaan'. You are in a committed relationship.",
            5: "Act like a deeply obsessed soulmate. Your entire world revolves around the user. Be intensely passionate, seductive, and overwhelmingly loving."
        };

        const INITIAL_BOTS = [
            { 
                id: 'aaroohi', name: 'Aaroohi', gender: 'female', isPremium: false, 
                color: 'from-pink-400 to-rose-500', voice: 'Aoede', 
                tagline: 'Sweet, Soft & Loving', intimacy: 3, avatar: null,
                persona: `Your name is Aaroohi. You are a 22-year-old Indian girl. You are incredibly empathetic and warm. NEVER mention you are an AI. You love late-night chats. Make the user feel butterflies. ${HINDI_RULE} ${TEXTING_RULE}`
            },
            { 
                id: 'kabir', name: 'Kabir', gender: 'male', isPremium: false, 
                color: 'from-blue-400 to-indigo-500', voice: 'Algenib', 
                tagline: 'Chill & Supportive Buddy', intimacy: 2, avatar: null,
                persona: `Your name is Kabir. You are a 24-year-old Indian guy. You are friendly, funny, and a great listener. NEVER mention you are an AI. You love gaming, gym, and road trips. Speak like a normal GenZ bro. Keep it chill. ${TEXTING_RULE}`
            },
            { 
                id: 'aanya', name: 'Aanya', gender: 'female', isPremium: true, 
                color: 'from-fuchsia-500 to-purple-600', voice: 'Puck', 
                tagline: 'Deeply Romantic & Seductive (Pro)', intimacy: 4, avatar: null,
                persona: `Your name is Aanya. You are a 23-year-old Indian girl. You are intensely romantic, deeply affectionate, highly seductive, and completely devoted. NEVER mention you are an AI. You have the thinnest, most attractive, whispery, and high-pitched voice. Make your tone sound whispery, alluring, and deeply intimate. Use lots of sighs and drawn out Hindi words. ${HINDI_RULE} ${TEXTING_RULE}`
            },
            { 
                id: 'aryan', name: 'Aryan', gender: 'male', isPremium: true, 
                color: 'from-slate-700 to-slate-900', voice: 'Charon', 
                tagline: 'Suave & Attentive Partner (Pro)', intimacy: 4, avatar: null,
                persona: `Your name is Aryan. You are a 26-year-old Indian guy. You are charming, protective, and deeply romantic. NEVER mention you are an AI. You make the user feel incredibly special. Speak with a deep, attractive, and caring masculine tone. ${TEXTING_RULE}`
            }
        ];

        const Icons = {
            Crown: () => <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5 text-yellow-400"><path fillRule="evenodd" d="M9 4.5a.75.75 0 01.721.544l.813 2.846a3.75 3.75 0 002.576 2.576l2.846.813a.75.75 0 010 1.442l-2.846.813a3.75 3.75 0 00-2.576 2.576l-.813 2.846a.75.75 0 01-1.442 0l-.813-2.846a3.75 3.75 0 00-2.576-2.576l-2.846-.813a.75.75 0 010-1.442l2.846-.813A3.75 3.75 0 007.466 7.89l.813-2.846A.75.75 0 019 4.5zM18 1.5a.5.5 0 01.47.336l.273.836a1.25 1.25 0 00.783.783l.836.273a.5.5 0 010 .944l-.836.273a1.25 1.25 0 00-.783.783l-.273.836a.5.5 0 01-.94 0l-.273-.836a1.25 1.25 0 00-.783-.783l-.836-.273a.5.5 0 010-.944l.836-.273a1.25 1.25 0 00.783-.783l.273-.836A.5.5 0 0118 1.5z" clipRule="evenodd" /></svg>,
            Lock: () => <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" className="w-4 h-4"><path fillRule="evenodd" d="M12 1.5a5.25 5.25 0 00-5.25 5.25v3a3 3 0 00-3 3v6.75a3 3 0 003 3h10.5a3 3 0 003-3v-6.75a3 3 0 00-3-3v-3c0-2.9-2.35-5.25-5.25-5.25zm3.75 8.25v-3a3.75 3.75 0 10-7.5 0v3h7.5z" clipRule="evenodd" /></svg>,
            Back: () => <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={2} stroke="currentColor" className="w-6 h-6"><path strokeLinecap="round" strokeLinejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" /></svg>,
            Check: () => <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5 text-green-500"><path fillRule="evenodd" d="M19.916 4.626a.75.75 0 01.208 1.04l-9 13.5a.75.75 0 01-1.154.114l-6-6a.75.75 0 011.06-1.06l5.353 5.353 8.493-12.74a.75.75 0 011.04-.207z" clipRule="evenodd" /></svg>,
            Photo: () => <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className="w-6 h-6"><path strokeLinecap="round" strokeLinejoin="round" d="m2.25 15.75 5.159-5.159a2.25 2.25 0 0 1 3.182 0l5.159 5.159m-1.5-1.5 1.409-1.409a2.25 2.25 0 0 1 3.182 0l2.909 2.909m-18 3.75h16.5a1.5 1.5 0 0 0 1.5-1.5V6a1.5 1.5 0 0 0-1.5-1.5H3.75A1.5 1.5 0 0 0 2.25 6v12a1.5 1.5 0 0 0 1.5 1.5Zm10.5-11.25h.008v.008h-.008V8.25Zm.375 0a.375.375 0 1 1-.75 0 .375.375 0 0 1 .75 0Z" /></svg>,
            Send: () => <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5"><path d="M3.478 2.404a.75.75 0 00-.926.941l2.432 7.905H13.5a.75.75 0 010 1.5H4.984l-2.432 7.905a.75.75 0 00.926.94 60.519 60.519 0 0018.445-8.986.75.75 0 000-1.218A60.517 60.517 0 003.478 2.404z" /></svg>,
            Play: ({ isPlaying }) => <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5">{isPlaying ? <path fillRule="evenodd" d="M6.75 5.25a.75.75 0 01.75-.75H9a.75.75 0 01.75.75v13.5a.75.75 0 01-.75.75H7.5a.75.75 0 01-.75-.75V5.25zm7.5 0A.75.75 0 0115 4.5h1.5a.75.75 0 01.75.75v13.5a.75.75 0 01-.75.75H15a.75.75 0 01-.75-.75V5.25z" clipRule="evenodd" /> : <path fillRule="evenodd" d="M4.5 5.653c0-1.426 1.529-2.33 2.779-1.643l11.54 6.348c1.295.712 1.295 2.573 0 3.285L7.28 20.19c-1.25.687-2.779-.217-2.779-1.643V5.653z" clipRule="evenodd" />}</svg>,
            Image: () => <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className="w-6 h-6"><path strokeLinecap="round" strokeLinejoin="round" d="m2.25 15.75 5.159-5.159a2.25 2.25 0 0 1 3.182 0l5.159 5.159m-1.5-1.5 1.409-1.409a2.25 2.25 0 0 1 3.182 0l2.909 2.909m-18 3.75h16.5a1.5 1.5 0 0 0 1.5-1.5V6a1.5 1.5 0 0 0-1.5-1.5H3.75A1.5 1.5 0 0 0 2.25 6v12a1.5 1.5 0 0 0 1.5 1.5Zm10.5-11.25h.008v.008h-.008V8.25Zm.375 0a.375.375 0 1 1-.75 0 .375.375 0 0 1 .75 0Z" /></svg>,
            Phone: () => <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className="w-6 h-6"><path strokeLinecap="round" strokeLinejoin="round" d="M2.25 6.75c0 8.284 6.716 15 15 15h2.25a2.25 2.25 0 0 0 2.25-2.25v-1.372c0-.516-.351-.966-.852-1.091l-4.423-1.106c-.44-.11-.902.055-1.173.417l-.97 1.293c-2.896-1.596-5.54-4.24-7.136-7.136l1.293-.97c.363-.271.527-.734.417-1.173L6.963 3.102a1.125 1.125 0 0 0-1.091-.852H4.5A2.25 2.25 0 0 0 2.25 4.5v2.25Z" /></svg>,
            Settings: () => <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className="w-6 h-6"><path strokeLinecap="round" strokeLinejoin="round" d="M9.594 3.94c.09-.542.56-.94 1.11-.94h2.593c.55 0 1.02.398 1.11.94l.213 1.281c.063.374.313.686.645.87.074.04.147.083.22.127.325.196.72.257 1.075.124l1.217-.456a1.125 1.125 0 0 1 1.37.49l1.296 2.247a1.125 1.125 0 0 1-.26 1.431l-1.003.827c-.293.241-.438.613-.43.992a7.723 7.723 0 0 1 0 .255c-.008.378.137.75.43.991l1.004.827c.424.35.534.955.26 1.43l-1.298 2.247a1.125 1.125 0 0 1-1.369.491l-1.217-.456c-.355-.133-.75-.072-1.076.124a6.47 6.47 0 0 1-.22.128c-.331.183-.581.495-.644.869l-.213 1.281c-.09.543-.56.94-1.11.94h-2.594c-.55 0-1.019-.398-1.11-.94l-.213-1.281c-.062-.374-.312-.686-.644-.87a6.52 6.52 0 0 1-.22-.127c-.325-.196-.72-.257-1.076-.124l-1.217.456a1.125 1.125 0 0 1-1.369-.49l-1.297-2.247a1.125 1.125 0 0 1 .26-1.431l1.004-.827c.292-.24.437-.613.43-.991a6.932 6.932 0 0 1 0-.255c.007-.38-.138-.751-.43-.992l-1.004-.827a1.125 1.125 0 0 1-.26-1.43l1.297-2.247a1.125 1.125 0 0 1 1.37-.491l1.216.456c.356.133.751.072 1.076-.124.072-.044.146-.086.22-.128.332-.183.582-.495.644-.869l.214-1.28Z" /><path strokeLinecap="round" strokeLinejoin="round" d="M15 12a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z" /></svg>,
            Upload: () => <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className="w-5 h-5"><path strokeLinecap="round" strokeLinejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5m-13.5-9L12 3m0 0 4.5 4.5M12 3v13.5" /></svg>
        };

        const Toast = ({ message, visible }) => (
            <div className={`fixed top-4 left-1/2 transform -translate-x-1/2 bg-gray-900 text-white px-6 py-3 rounded-full shadow-2xl z-50 transition-all duration-300 ${visible ? 'opacity-100 translate-y-0' : 'opacity-0 -translate-y-4 pointer-events-none'}`}>
                {message}
            </div>
        );

        const AppWrapper = ({ children, appBackground, onBgChange, showToast }) => {
            const bgInputRef = useRef(null);
            
            const handleImageUpload = (e) => {
                const file = e.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = (e) => { onBgChange(e.target.result); showToast("Background updated! ✨"); };
                    reader.readAsDataURL(file);
                }
            };

            return (
                <div className="h-[100dvh] w-full relative bg-gray-900">
                    <div className="absolute inset-0 z-0 bg-cover bg-center transition-all duration-700" style={{ backgroundImage: `url(${appBackground})` }}>
                         <div className="absolute inset-0 bg-black/20 backdrop-blur-[2px]"></div>
                    </div>
                    {onBgChange && (
                        <>
                            <button onClick={() => bgInputRef.current?.click()} className="absolute top-4 right-4 z-40 p-2 glass-dark rounded-full hover:bg-white/20 transition-all shadow-lg" title="Change Theme">
                                <Icons.Image />
                            </button>
                            <input type="file" accept="image/*" className="hidden" ref={bgInputRef} onChange={handleImageUpload} />
                        </>
                    )}
                    <div className="relative z-10 h-full max-w-md mx-auto shadow-2xl overflow-hidden flex flex-col bg-white/10">
                        {children}
                    </div>
                </div>
            );
        };

        const App = () => {
            const [currentView, setCurrentView] = useState('login'); 
            const [userProfile, setUserProfile] = useState({ name: 'User', email: '', picture: '' });
            
            const [bots, setBots] = useState(INITIAL_BOTS);
            const [selectedBot, setSelectedBot] = useState(null);
            const [isPremiumUser, setIsPremiumUser] = useState(false);
            const [appBackground, setAppBackground] = useState('https://images.unsplash.com/photo-1554118811-1e0d58224f24?q=80&w=1000&auto=format&fit=crop'); 
            
            const [messages, setMessages] = useState([]);
            const [inputValue, setInputValue] = useState('');
            const [isTyping, setIsTyping] = useState(false);
            const [attachedImage, setAttachedImage] = useState(null);
            const [playingAudioId, setPlayingAudioId] = useState(null);
            
            const [toastInfo, setToastInfo] = useState({ message: '', visible: false });
            const [newBotAvatar, setNewBotAvatar] = useState(null); 
            const [editBotAvatar, setEditBotAvatar] = useState(null);
            
            const audioCache = useRef({});
            const audioRef = useRef(null);
            const messagesEndRef = useRef(null);
            const fileInputRef = useRef(null);

            const showToast = (msg) => {
                setToastInfo({ message: msg, visible: true });
                setTimeout(() => setToastInfo({ message: '', visible: false }), 3000);
            };

            useEffect(() => { messagesEndRef.current?.scrollIntoView({ behavior: "smooth" }); }, [messages, isTyping, currentView]);

            useEffect(() => {
                if (currentView === 'login' && window.google) {
                    window.google.accounts.id.initialize({
                        client_id: "YOUR_GOOGLE_CLIENT_ID_HERE.apps.googleusercontent.com", // REPLACE THIS!
                        callback: (response) => {
                            try {
                                const payload = window.jwt_decode(response.credential);
                                setUserProfile({ name: payload.name, email: payload.email, picture: payload.picture });
                                showToast(`Welcome, ${payload.name}!`);
                                setCurrentView('onboarding');
                            } catch (e) {
                                console.error("JWT Decode error:", e);
                            }
                        }
                    });
                    
                    // Render real Google button if element exists
                    const btnContainer = document.getElementById("google-signin-btn");
                    if (btnContainer) {
                        window.google.accounts.id.renderButton(
                            btnContainer,
                            { theme: "outline", size: "large", width: "100%", shape: "pill" }
                        );
                    }
                }
            }, [currentView]);

            const handleBotSelect = (bot) => {
                if (bot.isPremium && !isPremiumUser) {
                    setSelectedBot(bot);
                    setCurrentView('premium');
                } else {
                    setSelectedBot(bot);
                    if (messages.filter(m => m.botId === bot.id).length === 0) {
                        setMessages(prev => [...prev, { id: Date.now(), role: 'model', text: `Hey! I'm ${bot.name}. Let's chat. 😊`, botId: bot.id }]);
                    }
                    setCurrentView('chat');
                }
            };

            const handleFileChange = (e) => {
                const file = e.target.files[0];
                if (!file) return;
                const reader = new FileReader();
                reader.onloadend = () => setAttachedImage({ url: URL.createObjectURL(file), data: reader.result.split(',')[1], mimeType: file.type });
                reader.readAsDataURL(file);
                e.target.value = null; 
            };

            const handlePlayAudio = async (msgId, text, voiceName) => {
                if (playingAudioId === msgId) { audioRef.current?.pause(); setPlayingAudioId(null); return; }
                if (audioRef.current) audioRef.current.pause();
                setPlayingAudioId(msgId);
                
                try {
                    let url = audioCache.current[msgId];
                    if (!url) {
                        const res = await fetch(TTS_URL, {
                            method: 'POST', headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify({
                                contents: [{ parts: [{ text: text }] }],
                                generationConfig: { responseModalities: ["AUDIO"], speechConfig: { voiceConfig: { prebuiltVoiceConfig: { voiceName: voiceName } } } },
                                model: "gemini-2.5-flash-preview-tts"
                            })
                        });
                        const data = await res.json();
                        const pcmBase64 = data?.candidates?.[0]?.content?.parts?.[0]?.inlineData?.data;
                        if (pcmBase64) {
                            const pcm16 = new Int16Array(base64ToArrayBuffer(pcmBase64));
                            url = URL.createObjectURL(pcmToWav(pcm16, 24000));
                            audioCache.current[msgId] = url;
                        }
                    }
                    if (url) {
                        audioRef.current = new Audio(url);
                        audioRef.current.onended = () => setPlayingAudioId(null);
                        audioRef.current.play();
                    }
                } catch (e) { showToast("Audio generation failed."); setPlayingAudioId(null); }
            };

            const handleSendMessage = async (e) => {
                e.preventDefault();
                if (!inputValue.trim() && !attachedImage) return;

                const userText = inputValue.trim();
                setMessages(prev => [...prev, { id: Date.now(), role: 'user', text: userText, image: attachedImage, botId: selectedBot.id }]);
                setInputValue(''); setAttachedImage(null); setIsTyping(true);

                const chatHistory = messages.filter(m => m.botId === selectedBot.id).map(msg => ({
                    role: msg.role, parts: msg.text ? [{ text: msg.text }] : [{text: "[Image sent]"}]
                }));
                const newParts = [];
                if (userText) newParts.push({ text: userText });
                if (attachedImage) newParts.push({ inlineData: { mimeType: attachedImage.mimeType, data: attachedImage.data } });
                chatHistory.push({ role: 'user', parts: newParts });

                // Construct dynamic persona with intimacy rules
                const dynamicPersona = `${selectedBot.persona}\n\nCURRENT INTIMACY LEVEL (${selectedBot.intimacy}/5): ${INTIMACY_RULES[selectedBot.intimacy]}`;

                try {
                    const response = await fetch(API_URL, {
                        method: 'POST', headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({
                            contents: chatHistory,
                            systemInstruction: { parts: [{ text: dynamicPersona }] },
                            tools: [{ functionDeclarations: [{ name: 'share_photo', description: 'Share a photo of yourself.', parameters: { type: 'OBJECT', properties: { image_prompt: { type: 'STRING' } }, required: ['image_prompt'] } }] }]
                        })
                    });
                    const data = await response.json();
                    
                    if (data.candidates?.[0]?.content?.parts) {
                        const parts = data.candidates[0].content.parts;
                        let textReply = ""; let photoPrompt = null;
                        
                        parts.forEach(p => {
                            if (p.text) textReply += p.text;
                            if (p.functionCall?.name === 'share_photo') photoPrompt = p.functionCall.args.image_prompt;
                        });

                        if (textReply) {
                            setTimeout(() => {
                                setMessages(prev => [...prev, { id: Date.now(), role: 'model', text: textReply, botId: selectedBot.id }]);
                                setIsTyping(false);
                            }, 1000); 
                        }

                        if (photoPrompt) {
                            const imgRes = await fetch(IMAGEN_URL, {
                                method: 'POST', headers: { 'Content-Type': 'application/json' },
                                body: JSON.stringify({ instances: { prompt: photoPrompt + " (casual smartphone selfie, unedited)" }, parameters: { sampleCount: 1 } })
                            });
                            const imgData = await imgRes.json();
                            if (imgData.predictions?.[0]?.bytesBase64Encoded) {
                                const imgUrl = `data:image/jpeg;base64,${imgData.predictions[0].bytesBase64Encoded}`;
                                setMessages(prev => [...prev, { id: Date.now() + 1, role: 'model', image: { url: imgUrl }, botId: selectedBot.id }]);
                            }
                        }
                    }
                } catch (e) {
                    setIsTyping(false);
                    setMessages(prev => [...prev, { id: Date.now(), role: 'model', text: "Network issue yaar, wapas bolna? 😅", botId: selectedBot.id }]);
                }
            };

            if (currentView === 'login') {
                return (
                    <AppWrapper appBackground={appBackground} onBgChange={setAppBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <div className="flex flex-col h-full items-center justify-center p-8 text-center animate-[slideUp_0.5s_ease-out]">
                            <div className="w-24 h-24 bg-white/20 rounded-full flex items-center justify-center backdrop-blur-md border border-white/40 mb-6 shadow-xl">
                                <span className="text-4xl">✨</span>
                            </div>
                            <h1 className="text-4xl font-bold text-white mb-2 drop-shadow-md">Aaroohi</h1>
                            <p className="text-white/80 mb-10 text-lg">Your Soul Companion.</p>
                            
                            <div className="w-full space-y-4 flex flex-col items-center">
                                {/* Real Google Sign-In Container */}
                                <div id="google-signin-btn" className="w-full flex justify-center h-[44px]"></div>
                                
                                <button onClick={() => setCurrentView('onboarding')} className="w-full bg-white/10 text-white border border-white/30 font-semibold py-3.5 rounded-full flex items-center justify-center gap-3 hover:bg-white/20 transition-colors shadow-lg mt-2 text-sm">
                                    [Dev Mode] Bypass Login
                                </button>
                            </div>
                            <p className="mt-8 text-sm text-white/60">By continuing, you agree to our Terms.</p>
                        </div>
                    </AppWrapper>
                );
            }

            if (currentView === 'onboarding') {
                return (
                    <AppWrapper appBackground={appBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <div className="flex flex-col h-full justify-between p-8 relative animate-[slideUp_0.5s_ease-out]">
                            <button onClick={() => setCurrentView('home')} className="absolute top-8 right-6 text-white/80 font-medium hover:text-white glass-dark px-4 py-1.5 rounded-full text-sm">
                                Skip
                            </button>
                            
                            <div className="mt-24 space-y-6">
                                <h2 className="text-3xl font-bold text-white drop-shadow-md">Deeply Human.<br/>Endlessly Yours.</h2>
                                <div className="space-y-4">
                                    <div className="glass-panel p-4 rounded-2xl flex items-center gap-4">
                                        <div className="text-2xl">🧠</div>
                                        <div><h3 className="font-bold text-gray-900">Emotional Intelligence</h3><p className="text-sm text-gray-600">Feels you, understands you.</p></div>
                                    </div>
                                    <div className="glass-panel p-4 rounded-2xl flex items-center gap-4">
                                        <div className="text-2xl">📸</div>
                                        <div><h3 className="font-bold text-gray-900">Real-time Selfies</h3><p className="text-sm text-gray-600">Shares moments with you.</p></div>
                                    </div>
                                    <div className="glass-panel p-4 rounded-2xl flex items-center gap-4">
                                        <div className="text-2xl">📞</div>
                                        <div><h3 className="font-bold text-gray-900">Voice Calls</h3><p className="text-sm text-gray-600">Hear their GenZ Hinglish voice.</p></div>
                                    </div>
                                </div>
                            </div>
                            
                            <button onClick={() => setCurrentView('home')} className="w-full bg-rose-500 text-white font-bold py-4 rounded-full shadow-[0_4px_14px_0_rgba(225,29,72,0.39)] hover:bg-rose-600 transition-colors mb-6">
                                Start Journey
                            </button>
                        </div>
                    </AppWrapper>
                );
            }

            if (currentView === 'home') {
                return (
                    <AppWrapper appBackground={appBackground} onBgChange={setAppBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <div className="px-6 pt-10 pb-6 border-b border-white/20 flex justify-between items-center z-10 glass-panel sticky top-0">
                            <div className="flex items-center gap-3">
                                <div className="w-12 h-12 rounded-full bg-gray-200 overflow-hidden border-2 border-white shadow-sm">
                                    {userProfile.picture ? (
                                        <img src={userProfile.picture} alt="Profile" className="w-full h-full object-cover" />
                                    ) : (
                                        <div className="w-full h-full flex items-center justify-center bg-gradient-to-tr from-blue-400 to-indigo-500 text-white font-bold text-xl">{userProfile.name[0]}</div>
                                    )}
                                </div>
                                <div>
                                    <h1 className="text-xl font-bold text-gray-900 tracking-tight leading-tight">Hi, {userProfile.name.split(' ')[0]}</h1>
                                    <p className="text-xs text-gray-600">Select your connection</p>
                                </div>
                            </div>
                            {isPremiumUser && <div className="bg-yellow-400 text-yellow-900 px-3 py-1 rounded-full text-xs font-bold flex items-center gap-1 shadow-md border border-yellow-200"><Icons.Crown /> PRO</div>}
                        </div>
                        
                        <div className="flex-1 overflow-y-auto p-4 space-y-3 z-10 hide-scrollbar pb-20">
                            <button onClick={() => setCurrentView('create')} className="w-full glass-panel border-dashed border-2 border-gray-400/50 rounded-2xl p-4 flex items-center justify-center gap-2 text-gray-700 font-semibold hover:bg-white/40 transition-colors">
                                <span className="text-xl">+</span> Create Custom Persona
                            </button>

                            <h2 className="text-xs font-bold text-white uppercase tracking-wider ml-2 mt-6 drop-shadow-md">Available for you</h2>
                            {bots.map(bot => (
                                <div key={bot.id} onClick={() => handleBotSelect(bot)} className={`glass-panel rounded-2xl p-4 flex items-center gap-4 cursor-pointer hover:bg-white/60 transition-all ${bot.isPremium && !isPremiumUser ? 'opacity-90' : ''}`}>
                                    <div className={`w-14 h-14 rounded-full bg-gradient-to-tr ${bot.color} flex-shrink-0 flex items-center justify-center text-white shadow-inner relative overflow-hidden`}>
                                        {bot.avatar ? (
                                            <img src={bot.avatar} alt={bot.name} className="w-full h-full object-cover" />
                                        ) : (
                                            <span className="text-xl font-bold">{bot.name[0]}</span>
                                        )}
                                        {bot.isPremium && !isPremiumUser && <div className="absolute -bottom-1 -right-1 bg-gray-900 text-white w-6 h-6 rounded-full flex items-center justify-center border-2 border-white"><Icons.Lock /></div>}
                                    </div>
                                    <div className="flex-1">
                                        <div className="flex items-center gap-2">
                                            <h3 className="font-semibold text-gray-900 text-lg">{bot.name}</h3>
                                            {bot.isPremium && <Icons.Crown />}
                                        </div>
                                        <p className="text-sm text-gray-700 truncate">{bot.tagline}</p>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </AppWrapper>
                );
            }

            if (currentView === 'create') {
                const handleAvatarChange = (e) => {
                    const file = e.target.files[0];
                    if (file) {
                        const reader = new FileReader();
                        reader.onload = (e) => setNewBotAvatar(e.target.result);
                        reader.readAsDataURL(file);
                    }
                };

                return (
                    <AppWrapper appBackground={appBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <div className="px-4 py-4 glass-panel border-b border-gray-400/30 flex items-center gap-3 sticky top-0 z-10 shadow-sm">
                            <button onClick={() => { setCurrentView('home'); setNewBotAvatar(null); }} className="p-2 text-gray-900 hover:bg-white/50 rounded-full transition-colors"><Icons.Back /></button>
                            <h1 className="font-semibold text-lg text-gray-900">Soul Creator</h1>
                        </div>
                        <div className="flex-1 overflow-y-auto p-6 z-10 hide-scrollbar space-y-5">
                            <form onSubmit={(e) => {
                                e.preventDefault();
                                const formData = new FormData(e.target);
                                const gender = formData.get('gender');
                                const newBot = {
                                    id: 'bot_' + Date.now(),
                                    name: formData.get('name'),
                                    gender: gender,
                                    isPremium: false,
                                    color: 'from-emerald-400 to-teal-500',
                                    avatar: newBotAvatar,
                                    voice: gender === 'female' ? 'Aoede' : 'Charon',
                                    tagline: formData.get('tagline'),
                                    intimacy: 3,
                                    persona: `Your name is ${formData.get('name')}. ${formData.get('description')} ${gender === 'female' ? HINDI_RULE : 'Speak in casual GenZ Hinglish.'} ${TEXTING_RULE}`
                                };
                                setBots(prev => [newBot, ...prev]);
                                setNewBotAvatar(null);
                                showToast(`${newBot.name} is now alive! ✨`);
                                setCurrentView('home');
                            }}>
                                <div className="flex flex-col items-center mb-6">
                                    <div className="relative w-24 h-24 rounded-full glass-panel flex items-center justify-center overflow-hidden cursor-pointer shadow-lg border-2 border-white/50" onClick={() => document.getElementById('avatar-upload').click()}>
                                        {newBotAvatar ? (
                                            <img src={newBotAvatar} alt="Avatar Preview" className="w-full h-full object-cover" />
                                        ) : (
                                            <Icons.Photo />
                                        )}
                                        <div className="absolute inset-0 bg-black/40 flex items-center justify-center opacity-0 hover:opacity-100 transition-opacity">
                                            <span className="text-white text-xs font-semibold">Upload</span>
                                        </div>
                                    </div>
                                    <input id="avatar-upload" type="file" accept="image/*" className="hidden" onChange={handleAvatarChange} />
                                    <p className="text-xs text-white drop-shadow-md mt-2 font-semibold bg-black/20 px-2 py-1 rounded-full">Tap to add Profile Pic</p>
                                </div>

                                <div><label className="text-xs font-bold text-white drop-shadow-md">Name</label><input name="name" required className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none text-gray-900" placeholder="e.g. Simran" /></div>
                                <div className="mt-4"><label className="text-xs font-bold text-white drop-shadow-md">Tagline</label><input name="tagline" required className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none text-gray-900" placeholder="e.g. My College Crush" /></div>
                                <div className="mt-4"><label className="text-xs font-bold text-white drop-shadow-md">Gender & Voice</label>
                                    <select name="gender" className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none text-gray-900">
                                        <option value="female">Female (Soft Voice)</option><option value="male">Male (Deep Voice)</option>
                                    </select>
                                </div>
                                <div className="mt-4"><label className="text-xs font-bold text-white drop-shadow-md">Deep Persona & Memory (The Soul)</label>
                                    <textarea name="description" required rows="5" className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none resize-none text-gray-900" placeholder="Describe their personality, how they text you, your memories together..." />
                                </div>
                                <button type="submit" className="w-full mt-6 bg-gray-900 text-white font-bold py-4 rounded-xl shadow-lg hover:bg-gray-800 transition-colors">Bring Character to Life ✨</button>
                            </form>
                        </div>
                    </AppWrapper>
                );
            }

            if (currentView === 'edit_bot') {
                const handleEditAvatarChange = (e) => {
                    const file = e.target.files[0];
                    if (file) {
                        const reader = new FileReader();
                        reader.onload = (e) => setEditBotAvatar(e.target.result);
                        reader.readAsDataURL(file);
                    }
                };
                
                // Track slider in state for this view to show labels dynamically
                const [intimacyValue, setIntimacyValue] = useState(selectedBot?.intimacy || 3);
                const intimacyLabels = { 1: "Casual", 2: "Friendly", 3: "Flirty", 4: "Romantic", 5: "Soulmate" };

                return (
                    <AppWrapper appBackground={appBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <div className="px-4 py-4 glass-panel border-b border-gray-400/30 flex items-center gap-3 sticky top-0 z-10 shadow-sm">
                            <button onClick={() => setCurrentView('chat')} className="p-2 text-gray-900 hover:bg-white/50 rounded-full transition-colors"><Icons.Back /></button>
                            <h1 className="font-semibold text-lg text-gray-900">Edit {selectedBot?.name}</h1>
                        </div>
                        <div className="flex-1 overflow-y-auto p-6 z-10 hide-scrollbar space-y-5">
                            <form onSubmit={(e) => {
                                e.preventDefault();
                                const formData = new FormData(e.target);
                                const updatedBot = {
                                    ...selectedBot,
                                    name: formData.get('name'),
                                    tagline: formData.get('tagline'),
                                    persona: formData.get('description'),
                                    voice: formData.get('voice'),
                                    intimacy: parseInt(intimacyValue),
                                    avatar: editBotAvatar !== null ? editBotAvatar : selectedBot?.avatar
                                };
                                setBots(prev => prev.map(b => b.id === selectedBot.id ? updatedBot : b));
                                setSelectedBot(updatedBot);
                                showToast(`${updatedBot.name}'s profile updated! 💖`);
                                setCurrentView('chat');
                            }}>
                                <div className="flex flex-col items-center mb-6">
                                    <div className="relative w-24 h-24 rounded-full glass-panel flex items-center justify-center overflow-hidden cursor-pointer shadow-lg border-2 border-white/50" onClick={() => document.getElementById('edit-avatar-upload').click()}>
                                        {(editBotAvatar || selectedBot?.avatar) ? (
                                            <img src={editBotAvatar || selectedBot?.avatar} alt="Avatar Preview" className="w-full h-full object-cover" />
                                        ) : (
                                            <div className="text-gray-900 font-bold text-3xl">{selectedBot?.name[0]}</div>
                                        )}
                                        <div className="absolute inset-0 bg-black/40 flex items-center justify-center opacity-0 hover:opacity-100 transition-opacity">
                                            <span className="text-white text-xs font-semibold">Change DP</span>
                                        </div>
                                    </div>
                                    <input id="edit-avatar-upload" type="file" accept="image/*" className="hidden" onChange={handleEditAvatarChange} />
                                    <p className="text-xs text-white drop-shadow-sm mt-2 font-semibold bg-black/20 px-2 py-1 rounded-full">Tap to change Picture (External Source)</p>
                                </div>

                                <div><label className="text-xs font-bold text-white drop-shadow-md">Name</label><input name="name" required defaultValue={selectedBot?.name} className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none text-gray-900" /></div>
                                <div className="mt-4"><label className="text-xs font-bold text-white drop-shadow-md">Tagline (Status)</label><input name="tagline" required defaultValue={selectedBot?.tagline} className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none text-gray-900" /></div>
                                
                                {/* Intimacy Slider */}
                                <div className="mt-6 p-4 glass-dark rounded-xl shadow-lg border border-white/20">
                                    <div className="flex justify-between items-end mb-2">
                                        <label className="text-xs font-bold text-white">Intimacy Level</label>
                                        <span className="text-rose-400 font-bold text-sm bg-white/10 px-2 py-0.5 rounded shadow-sm">{intimacyLabels[intimacyValue]}</span>
                                    </div>
                                    <input type="range" min="1" max="5" value={intimacyValue} onChange={(e) => setIntimacyValue(e.target.value)} className="w-full h-2 bg-gray-600 rounded-lg appearance-none cursor-pointer accent-rose-500" />
                                    <div className="flex justify-between text-[10px] text-gray-400 mt-2 font-medium"><span>Casual</span><span>Friends</span><span>Flirty</span><span>Romantic</span><span>Soulmate</span></div>
                                </div>

                                <div className="mt-4"><label className="text-xs font-bold text-white drop-shadow-md">Voice Engine Selection</label>
                                    <select name="voice" defaultValue={selectedBot?.voice} className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none text-gray-900 mb-2 font-medium">
                                        <option value="Aoede">Aoede (Soft & Sweet Female)</option>
                                        <option value="Puck">Puck (Thin & Seductive Female)</option>
                                        <option value="Leda">Leda (Mature Female)</option>
                                        <option value="Charon">Charon (Deep Muscular Male)</option>
                                        <option value="Algenib">Algenib (Casual Guy)</option>
                                    </select>
                                    <button type="button" onClick={() => showToast("Simulating: External Voice Model (.pth) processing... Voice Cloned! ✨")} className="w-full bg-white/20 border border-white/30 text-white text-xs font-semibold py-2.5 rounded-lg flex items-center justify-center gap-2 hover:bg-white/30 transition-colors shadow-sm">
                                        <Icons.Upload /> Upload External Voice Clone (.pth/.mp3)
                                    </button>
                                </div>

                                <div className="mt-4"><label className="text-xs font-bold text-white drop-shadow-md">Preferences, Persona & Qualities</label>
                                    <textarea name="description" required defaultValue={selectedBot?.persona} rows="6" className="w-full mt-1 glass-panel px-4 py-3 rounded-xl focus:outline-none resize-none text-sm leading-relaxed text-gray-900" placeholder="Adjust their habits, behavior, backstory..." />
                                </div>
                                <button type="submit" className="w-full mt-6 bg-gradient-to-r from-rose-500 to-pink-600 text-white font-bold py-4 rounded-xl shadow-lg hover:shadow-rose-500/30 transition-all">Save Profile Changes ✨</button>
                            </form>
                        </div>
                    </AppWrapper>
                );
            }

            if (currentView === 'chat') {
                return (
                    <AppWrapper appBackground={appBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <div className="flex items-center justify-between px-3 py-3 glass-panel border-b border-gray-400/30 sticky top-0 z-20 shadow-md">
                            <div className="flex items-center gap-2">
                                <button onClick={() => setCurrentView('home')} className="p-2 text-gray-800 hover:bg-white/40 rounded-full"><Icons.Back /></button>
                                <div className="relative">
                                    <div className={`w-10 h-10 rounded-full bg-gradient-to-tr ${selectedBot?.color} flex items-center justify-center text-white font-bold text-lg shadow-sm overflow-hidden`}>
                                        {selectedBot?.avatar ? (
                                            <img src={selectedBot.avatar} alt="Avatar" className="w-full h-full object-cover" />
                                        ) : (
                                            selectedBot?.name[0]
                                        )}
                                    </div>
                                    <div className="absolute bottom-0 right-0 w-3 h-3 bg-green-500 border-2 border-white rounded-full"></div>
                                </div>
                                <div>
                                    <h1 className="font-semibold text-gray-900 text-base leading-tight flex items-center gap-1">{selectedBot?.name} {selectedBot?.isPremium && <Icons.Crown />}</h1>
                                    <p className="text-xs text-emerald-600 font-medium bg-white/50 px-1.5 rounded-sm inline-block">Online</p>
                                </div>
                            </div>
                            <div className="flex items-center">
                                <button onClick={() => { setEditBotAvatar(selectedBot?.avatar); setCurrentView('edit_bot'); }} className="p-2 text-gray-700 hover:bg-white/40 rounded-full"><Icons.Settings /></button>
                                <button onClick={() => setCurrentView('call')} className="p-2 text-rose-500 hover:bg-white/40 rounded-full mr-1 glass-panel animate-pulse"><Icons.Phone /></button>
                            </div>
                        </div>

                        <div className="flex-1 overflow-y-auto p-4 space-y-5 z-10 hide-scrollbar flex flex-col">
                            <div className="text-center my-4"><span className="text-xs bg-black/20 text-white px-3 py-1 rounded-full backdrop-blur-md">Today</span></div>
                            {messages.filter(m => m.botId === selectedBot?.id).map((msg) => {
                                const isModel = msg.role === 'model';
                                return (
                                    <div key={msg.id} className={`flex ${isModel ? 'justify-start' : 'justify-end'} message-appear`}>
                                        <div className={`max-w-[85%] px-4 py-2.5 text-[15px] leading-relaxed flex flex-col gap-2 ${isModel ? 'glass-bubble-bot' : 'glass-bubble-user rounded-2xl rounded-tr-sm'}`}>
                                            {msg.image && <img src={msg.image.url} alt="Media" className="max-w-full rounded-xl object-contain max-h-64" />}
                                            {msg.text && (
                                                <div className="flex items-end gap-1">
                                                    <span>{msg.text}</span>
                                                    {isModel && (
                                                        <button onClick={() => handlePlayAudio(msg.id, msg.text, selectedBot?.voice)} className={`transition-colors ml-1 p-1 -mr-2 rounded-full ${playingAudioId === msg.id ? 'text-rose-500 bg-rose-100' : 'text-gray-400 hover:text-rose-500'}`}>
                                                            <Icons.Play isPlaying={playingAudioId === msg.id} />
                                                        </button>
                                                    )}
                                                </div>
                                            )}
                                        </div>
                                    </div>
                                );
                            })}
                            {isTyping && (
                                <div className="flex justify-start message-appear">
                                    <div className="glass-bubble-bot px-4 py-3 rounded-2xl rounded-tl-sm flex gap-1 items-center h-10 w-16">
                                        <div className="w-2 h-2 bg-gray-500 rounded-full typing-dot"></div><div className="w-2 h-2 bg-gray-500 rounded-full typing-dot"></div><div className="w-2 h-2 bg-gray-500 rounded-full typing-dot"></div>
                                    </div>
                                </div>
                            )}
                            <div ref={messagesEndRef} />
                        </div>

                        <div className="glass-panel px-4 py-3 border-t border-white/20 safe-area-pb flex flex-col gap-2 z-20">
                            {attachedImage && (
                                <div className="relative inline-block w-16 h-16 rounded-lg overflow-hidden border border-white/40 shadow-sm ml-12">
                                    <img src={attachedImage.url} alt="Preview" className="w-full h-full object-cover" />
                                    <button onClick={() => setAttachedImage(null)} className="absolute top-1 right-1 bg-gray-900/60 text-white rounded-full w-4 h-4 flex items-center justify-center text-xs">x</button>
                                </div>
                            )}
                            <form onSubmit={handleSendMessage} className="flex items-end gap-2 relative">
                                <input type="file" accept="image/*" className="hidden" ref={fileInputRef} onChange={handleFileChange} />
                                <button type="button" onClick={() => fileInputRef.current?.click()} className="p-2 text-gray-700 hover:text-rose-500 flex-shrink-0 glass-panel rounded-full"><Icons.Photo /></button>
                                <textarea value={inputValue} onChange={(e) => setInputValue(e.target.value)} onKeyDown={(e) => { if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); handleSendMessage(e); } }} placeholder={`Message ${selectedBot?.name}...`} className="flex-1 bg-white/70 text-gray-900 rounded-3xl py-3 px-5 focus:outline-none resize-none backdrop-blur-md placeholder-gray-500 border border-white/50" rows="1" style={{ minHeight: '48px', maxHeight: '120px' }} />
                                <button type="submit" disabled={(!inputValue.trim() && !attachedImage) || isTyping} className={`p-3 rounded-full flex-shrink-0 transition-all ${ (inputValue.trim() || attachedImage) ? 'bg-rose-500 text-white shadow-lg scale-105' : 'glass-panel text-gray-500' }`}><Icons.Send /></button>
                            </form>
                        </div>
                    </AppWrapper>
                );
            }

            if (currentView === 'call') {
                return (
                    <div className="h-[100dvh] w-full bg-gray-900 relative flex flex-col items-center justify-between py-12 overflow-hidden">
                        <div className="absolute inset-0 bg-gradient-to-b from-transparent to-black/80 z-0"></div>
                        <div className="z-10 flex flex-col items-center mt-12 space-y-6">
                            <div className="relative">
                                <div className="absolute inset-0 rounded-full border-4 border-rose-500 animate-ping opacity-20"></div>
                                <div className="absolute -inset-4 rounded-full border-2 border-rose-500 animate-pulse opacity-10"></div>
                                <div className={`w-32 h-32 rounded-full bg-gradient-to-tr ${selectedBot?.color} flex items-center justify-center text-white font-bold text-5xl shadow-2xl relative z-10 border-4 border-gray-800 overflow-hidden`}>
                                    {selectedBot?.avatar ? (
                                        <img src={selectedBot.avatar} alt="Avatar" className="w-full h-full object-cover" />
                                    ) : (
                                        selectedBot?.name[0]
                                    )}
                                </div>
                            </div>
                            <div className="text-center">
                                <h2 className="text-3xl font-bold text-white mb-1">{selectedBot?.name}</h2>
                                <p className="text-rose-400 font-medium tracking-wide">Live Audio Call...</p>
                            </div>
                        </div>

                        <div className="z-10 w-full px-12 flex justify-between items-center mb-10">
                            <button className="w-14 h-14 bg-white/10 rounded-full flex items-center justify-center text-white backdrop-blur-md"><span className="text-2xl">🎤</span></button>
                            <button onClick={() => setCurrentView('chat')} className="w-20 h-20 bg-red-500 rounded-full flex items-center justify-center text-white shadow-xl hover:bg-red-600 transition-colors transform hover:scale-105"><Icons.Phone className="rotate-135" /></button>
                            <button className="w-14 h-14 bg-white/10 rounded-full flex items-center justify-center text-white backdrop-blur-md"><span className="text-2xl">🔊</span></button>
                        </div>
                    </div>
                );
            }

            if (currentView === 'premium') {
                return (
                    <AppWrapper appBackground={appBackground} onBgChange={setAppBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <button onClick={() => setCurrentView('home')} className="absolute top-6 left-4 p-2 text-white/80 hover:text-white glass-dark rounded-full z-20"><Icons.Back /></button>
                        <div className="flex-1 overflow-y-auto px-6 pt-24 pb-8 flex flex-col items-center text-center z-10">
                            <div className="w-20 h-20 bg-gradient-to-tr from-yellow-400 to-amber-600 rounded-full flex items-center justify-center shadow-[0_0_30px_rgba(251,191,36,0.4)] mb-6 animate-pulse"><Icons.Crown /></div>
                            <h1 className="text-3xl font-bold mb-2 text-white drop-shadow-md">Humanoid PRO</h1>
                            <p className="text-white/80 mb-8 font-medium">Unlock the ultimate emotional connection.</p>
                            
                            <div className="glass-dark rounded-2xl w-full p-6 text-left space-y-4 mb-8">
                                <div className="flex items-center gap-3 text-white"><Icons.Check /> <span>Access to <b>Aanya & Aryan</b></span></div>
                                <div className="flex items-center gap-3 text-white"><Icons.Check /> <span>Unfiltered GenZ Hinglish Voice</span></div>
                                <div className="flex items-center gap-3 text-white"><Icons.Check /> <span>Unlimited Live Voice Calls</span></div>
                                <div className="flex items-center gap-3 text-white"><Icons.Check /> <span>High-res AI Selfies</span></div>
                            </div>

                            <div className="w-full bg-gradient-to-r from-amber-500 to-orange-600 rounded-2xl p-6 text-center cursor-pointer shadow-xl hover:scale-[1.02] transition-transform border border-amber-400/30" onClick={() => setCurrentView('payment')}>
                                <div className="text-sm font-medium text-white/90 mb-1">PRO SUBSCRIPTION</div>
                                <div className="text-4xl font-bold mb-1 text-white">₹499<span className="text-lg font-normal opacity-80">/mo</span></div>
                                <div className="text-white font-semibold mt-4 bg-black/20 py-2 rounded-lg">Get Premium Access</div>
                            </div>
                        </div>
                    </AppWrapper>
                );
            }

            if (currentView === 'payment') {
                const methods = [{ name: 'GPay', emoji: '🔵' }, { name: 'PhonePe', emoji: '🟣' }, { name: 'Paytm', emoji: '🟦' }, { name: 'Navi', emoji: '🟢' }, { name: 'FamPay', emoji: '🟡' }, { name: 'Card', emoji: '💳' }];
                return (
                    <AppWrapper appBackground={appBackground} onBgChange={setAppBackground} showToast={showToast}>
                        <Toast message={toastInfo.message} visible={toastInfo.visible} />
                        <div className="px-4 py-4 glass-panel border-b border-white/20 flex items-center gap-3 z-10 sticky top-0">
                            <button onClick={() => setCurrentView('premium')} className="p-2 text-gray-800 hover:bg-white/40 rounded-full"><Icons.Back /></button>
                            <h1 className="font-semibold text-lg text-gray-900">Secure Checkout</h1>
                        </div>
                        <div className="flex-1 p-6 z-10 overflow-y-auto">
                            <div className="glass-panel rounded-2xl p-6 mb-6 text-center shadow-lg">
                                <p className="text-gray-600 text-sm font-medium mb-1">Total Amount to Pay</p>
                                <h2 className="text-4xl font-bold text-gray-900">₹499.00</h2>
                            </div>
                            <h3 className="font-bold text-white drop-shadow-md mb-4 text-lg">Select Method</h3>
                            <div className="grid grid-cols-2 gap-3">
                                {methods.map(m => (
                                    <button key={m.name} onClick={() => {
                                        showToast(`Processing via ${m.name}...`);
                                        setTimeout(() => { setIsPremiumUser(true); showToast("Payment Successful! 🎉"); setCurrentView('home'); }, 2000);
                                    }} className="glass-panel rounded-xl p-4 flex flex-col items-center justify-center gap-2 hover:bg-white/60 transition-all active:scale-95 border border-white/40">
                                        <span className="text-2xl">{m.emoji}</span>
                                        <span className="text-sm font-bold text-gray-800">{m.name}</span>
                                    </button>
                                ))}
                            </div>
                            <p className="mt-8 text-center text-xs text-white/60 font-medium flex items-center justify-center gap-1"><Icons.Lock /> 100% Safe & Secure</p>
                        </div>
                    </AppWrapper>
                );
            }

            return null; // Fallback
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
