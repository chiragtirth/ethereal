<script lang="ts">
    // App.svelte -- basically index.html
    import Config from "./Config.svelte";
    import Proxy from "./Proxy.svelte";
    import config from "./config.svelte";
    import proxyManager from "./proxy.svelte";
    import { onEnterKeyPressed } from "./util";
    import autoProxyProber from "./prober.svelte";
    import { History } from "./history";
    import {
        Search,
        Settings2,
        RotateCw,
        ArrowRight,
        ArrowLeft,
        X,
        Gamepad2,
        Home,
        Globe,
        Monitor,
        Sun,
        Wifi
    } from "@lucide/svelte";

    $effect(() => {
        if (config.useBare && config.bareSelectedProxy === "auto") {
            autoProxyProber.probeBare();
        } else if (config.wispSelectedProxy === "auto") {
            autoProxyProber.probeWisp();
        }
    });

    let urlBar: HTMLInputElement = $state();

    $effect(() => {
        proxyManager.setProxyServer(proxyManager.proxyUrl);
    });

    let destinationInput = $state("");

    let isConfigOpen = $state(false);
    let disclaimerAccepted = $state(false);
    let isEmulatorOpen = $state(false);
    let showPasswordScreen = $state(false);
    let passwordInput = $state("");
    let passwordCorrect = $state(false);

    function acceptDisclaimer() {
        disclaimerAccepted = true;
    }

    function openEmulator() {
        showPasswordScreen = true;
    }

    function checkPassword() {
        if (passwordInput === "2612") {
            passwordCorrect = true;
            showPasswordScreen = false;
            isEmulatorOpen = true;
            passwordInput = "";
        } else {
            passwordInput = "";
            // You could add a visual feedback here for wrong password
        }
    }

    function cancelPassword() {
        showPasswordScreen = false;
        passwordInput = "";
    }

    function openAmazon() {
        destinationInput = 'https://amazon.com';
        startProxy();
    }

    function openChatGPT() {
        destinationInput = 'https://chat.openai.com';
        startProxy();
    }

    function openDiscord() {
        destinationInput = 'https://discord.com';
        startProxy();
    }

    function openGitHub() {
        destinationInput = 'https://github.com';
        startProxy();
    }

    function openGoogle() {
        destinationInput = 'https://google.com';
        startProxy();
    }

    function openPinterest() {
        destinationInput = 'https://pinterest.com';
        startProxy();
    }

    function openTwitch() {
        destinationInput = 'https://twitch.tv';
        startProxy();
    }

    function openYouTube() {
        destinationInput = 'https://youtube.com';
        startProxy();
    }

    function startProxy() {
        if (proxyManager.startProxy(destinationInput)) {
            destinationInput = "";
            console.log("proxy started")
        }
    }

    let iframeHasLoaded = $state(false);
    let isProxyOpen = $state(false)

    let iframe: HTMLIFrameElement = $state();

    let searchbar: HTMLInputElement = $state()
    let loader: HTMLSpanElement = $state()

    const iframeAllow =
        "accelerometer ambient-light-sensor attribution-reporting autoplay bluetooth browsing-topics camera compute-pressure " +
        "cross-origin-isolated display-capture document-domain encrypted-media fullscreen gamepad geolocation gyroscope hid " +
        "identity-credentials-get idle-detection local-fonts magnetometer microphone midi otp-credentials payment " +
        "picture-in-picture publickey-credentials-create publickey-credentials-get screen-wake-lock serial speaker-selection " +
        "storage-access usb web-share window-management xr-spatial-tracking";

    const iframeSandbox =
        "allow-popups allow-downloads allow-forms allow-modals allow-orientation-lock " +
        "allow-pointer-lock allow-presentation allow-same-origin allow-scripts allow-storage-access-by-user-activation";

    function onIframeLoad() {
        if (iframe == undefined) return;
        const src = iframe.contentWindow.location.pathname;
        if (!src.includes(proxyManager.uvConfig.prefix)) return;

        iframeHasLoaded = true;
                    
        if (searchbar) {
            searchbar.value = proxyManager.url
            console.log("Set Value: " + proxyManager.url)
        }

        proxyManager.url = proxyManager.uvConfig.decodeUrl(
            src.slice(proxyManager.uvConfig.prefix.length),
        );
    }

    let proxyHistory = new History();
    $effect(() => {
        proxyHistory.addToHistory(proxyManager.url);
    });

    $effect(() => {
        if (urlBar && !proxyManager.isProxyOpen) {
            urlBar.focus();
        }
    });

    function setUrl(url: string | null) {
        if (url == null) return;
        proxyManager.url = url;
        proxyManager.reloadIframe();
    }
</script>

{#if !disclaimerAccepted}
    <!-- Disclaimer Page -->
    <div class="min-h-screen bg-gradient-to-br from-gray-900 via-blue-900 to-purple-900 flex items-center justify-center p-4">
        <div class="max-w-2xl w-full bg-white/10 backdrop-blur-lg rounded-2xl shadow-2xl p-8 border border-white/20">
            <div class="text-center mb-8">
                <h1 class="text-4xl font-bold text-white mb-4">⚠️ Important Disclaimer</h1>
                <div class="w-16 h-1 bg-gradient-to-r from-blue-400 to-purple-400 mx-auto rounded-full"></div>
            </div>
            
            <div class="space-y-6 text-gray-200">
                <div class="bg-yellow-500/20 border border-yellow-500/30 rounded-lg p-4">
                    <h2 class="text-xl font-semibold text-yellow-300 mb-3">🚨 Legal Notice</h2>
                    <p class="text-sm leading-relaxed">
                        This proxy service is provided for educational and research purposes only. 
                        Users are responsible for complying with all applicable laws and regulations 
                        in their jurisdiction. If you are a school administrator and want this taken down, 
                        contact us at ethereal.proxy1@gmail.com
                    </p>
                </div>

                <div class="bg-red-500/20 border border-red-500/30 rounded-lg p-4">
                    <h2 class="text-xl font-semibold text-red-300 mb-3">🔒 Privacy & Security</h2>
                    <p class="text-sm leading-relaxed">
                        This service may log your activity for security purposes. 
                        Do not use this service for sensitive or confidential information. 
                        We are not responsible for any data breaches or security incidents.
                    </p>
                </div>

                <div class="bg-blue-500/20 border border-blue-500/30 rounded-lg p-4">
                    <h2 class="text-xl font-semibold text-blue-300 mb-3">⚖️ Terms of Use</h2>
                    <ul class="text-sm leading-relaxed space-y-2">
                        <li>• Do not use for illegal activities</li>
                        <li>• Respect website terms of service</li>
                        <li>• No guarantee of service availability</li>
                        <li>• Use at your own risk</li>
                    </ul>
                </div>

                <div class="bg-green-500/20 border border-green-500/30 rounded-lg p-4">
                    <h2 class="text-xl font-semibold text-green-300 mb-3">✅ Responsible Usage</h2>
                    <p class="text-sm leading-relaxed">
                        By continuing, you acknowledge that you have read and understood 
                        this disclaimer and agree to use this service responsibly.
                    </p>
                </div>
            </div>

            <div class="mt-8 flex flex-col sm:flex-row gap-4 justify-center">
                <button 
                    onclick={acceptDisclaimer}
                    class="px-8 py-3 bg-gradient-to-r from-blue-500 to-purple-500 text-white font-semibold rounded-lg hover:from-blue-600 hover:to-purple-600 transition-all duration-300 shadow-lg hover:shadow-xl transform hover:scale-105"
                >
                    I Understand & Accept
                </button>
                <button 
                    onclick={() => window.close()}
                    class="px-8 py-3 bg-gray-600 text-white font-semibold rounded-lg hover:bg-gray-700 transition-all duration-300 shadow-lg"
                >
                    Exit
                </button>
            </div>
        </div>
    </div>
{:else if showPasswordScreen}
    <!-- Password Screen -->
    <div class="min-h-screen bg-gradient-to-br from-gray-900 via-blue-900 to-purple-900 flex items-center justify-center p-4">
        <div class="max-w-md w-full bg-white/10 backdrop-blur-lg rounded-2xl shadow-2xl p-8 border border-white/20">
            <div class="text-center mb-8">
                <div class="w-16 h-16 bg-purple-500/20 rounded-full flex items-center justify-center mx-auto mb-4">
                    <Gamepad2 class="w-8 h-8 text-purple-300" />
                </div>
                <h1 class="text-3xl font-bold text-white mb-2">🔒 Emulator Access</h1>
                <p class="text-gray-300 text-sm">Enter password to access the emulator</p>
                <div class="w-16 h-1 bg-gradient-to-r from-purple-400 to-blue-400 mx-auto rounded-full mt-4"></div>
            </div>
            
            <div class="space-y-6">
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-300 mb-2">
                            Password
                        </label>
                        <input
                            type="password"
                            bind:value={passwordInput}
                            onkeydown={(e) => e.key === 'Enter' && checkPassword()}
                            class="w-full px-4 py-3 bg-white/10 border border-white/20 rounded-lg text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:border-transparent transition-all"
                            placeholder="Enter password..."
                            autofocus
                        />
                    </div>
                </div>

                <div class="flex flex-col sm:flex-row gap-3">
                    <button 
                        onclick={checkPassword}
                        class="flex-1 px-6 py-3 bg-gradient-to-r from-purple-500 to-blue-500 text-white font-semibold rounded-lg hover:from-purple-600 hover:to-blue-600 transition-all duration-300 shadow-lg hover:shadow-xl transform hover:scale-105"
                    >
                        Access Emulator
                    </button>
                    <button 
                        onclick={cancelPassword}
                        class="flex-1 px-6 py-3 bg-gray-600 text-white font-semibold rounded-lg hover:bg-gray-700 transition-all duration-300 shadow-lg"
                    >
                        Cancel
                    </button>
                </div>
            </div>
        </div>
    </div>
{:else if isEmulatorOpen}
    <!-- Emulator Interface -->
    <div class="w-full h-screen relative">
        <!-- Back Button - Positioned to not interfere with emulator controls -->
        <button
            class="absolute top-4 left-4 z-50 btn-circle bg-blue-500/80 backdrop-blur-sm p-3 text-sm cursor-pointer pointer-events-auto hover:brightness-75 transition-all shadow-lg"
            title="Back to Proxy"
            onclick={() => (isEmulatorOpen = false)}
        ><ArrowLeft class="scale-95" /></button>
        
        <!-- Full Screen Emulator -->
        <iframe
            title="Retro Game Emulator"
            class="w-full h-full border-0"
            src="https://demo.emulatorjs.org/"
            allow="gamepad; microphone; camera"
            sandbox="allow-same-origin allow-scripts allow-forms allow-popups allow-pointer-lock allow-downloads"
        ></iframe>
    </div>
    <Config bind:isConfigOpen></Config>
{:else if proxyManager.isProxyOpen}
    <div>
        <iframe
            bind:this={iframe}
            title="Proxy"
            class="w-full h-screen bg-transparent"
            src={proxyManager.iframeUrl}
            onload={onIframeLoad}
            allow={iframeAllow}
            sandbox={iframeSandbox}
        ></iframe>
    </div>
    <Config bind:isConfigOpen></Config>

{:else}
    <Config bind:isConfigOpen></Config>
    
    <!-- New Ambient-Style Homepage -->
    <div class="min-h-screen bg-black text-white">
        <!-- Top Navigation Bar with Integrated Controls -->
        <nav class="flex items-center justify-between px-6 py-4 border-b border-gray-800">
            <div class="flex items-center space-x-2">
                <div class="w-8 h-8 bg-white rounded-full flex items-center justify-center">
                    <div class="w-4 h-4 bg-black rounded-full"></div>
                </div>
                <div class="flex space-x-6 ml-6">
                    <button class="text-white hover:text-gray-300 transition-colors">Home</button>
                    <button class="text-white hover:text-gray-300 transition-colors">Games</button>
                    <button class="text-white hover:text-gray-300 transition-colors">Apps</button>
                    <button onclick={openEmulator} class="text-white hover:text-gray-300 transition-colors">Emulator</button>
                </div>
            </div>
            
            <!-- Integrated Toolbar Controls -->
            <div class="flex items-center space-x-4">
                <!-- Navigation Controls -->
                <div class="flex items-center space-x-2">
                    <button
                        class="w-8 h-8 bg-blue-500 rounded-full flex items-center justify-center hover:bg-blue-600 transition-colors"
                        title="Previous Page"
                        onclick={() => setUrl(proxyHistory.goBackward())}
                    ><ArrowLeft class="w-4 h-4" /></button>
                    <button
                        class="w-8 h-8 bg-blue-500 rounded-full flex items-center justify-center hover:bg-blue-600 transition-colors"
                        title="Next Page"
                        onclick={() => setUrl(proxyHistory.goForward())}
                    ><ArrowRight class="w-4 h-4" /></button>
                    <button
                        class="w-8 h-8 bg-blue-500 rounded-full flex items-center justify-center hover:bg-blue-600 transition-colors"
                        title="Refresh Page"
                        onclick={() => iframe?.contentWindow?.location?.reload()}
                    ><RotateCw class="w-4 h-4" /></button>
                </div>
                
                <!-- Search Bar -->
                <div class="flex items-center space-x-2">
                    <input
                        type="text"
                        class="px-4 py-2 bg-gray-800 border border-gray-700 rounded-lg text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all w-64"
                        placeholder="Search anything..."
                        onkeydown={onEnterKeyPressed(startProxy)}
                        bind:this={urlBar}
                        bind:value={destinationInput}
                    />
                    <button
                        onclick={startProxy}
                        disabled={proxyManager.proxyUrl === "" || !proxyManager.serviceWorker}
                        class="w-8 h-8 bg-blue-500 rounded-full flex items-center justify-center hover:bg-blue-600 transition-colors disabled:opacity-50"
                        title="Start proxy"
                    ><Search class="w-4 h-4" /></button>
                </div>
                
                <!-- Settings -->
                <button onclick={() => (isConfigOpen = true)} class="text-white hover:text-gray-300 transition-colors">
                    <Settings2 class="w-5 h-5" />
                </button>
            </div>
        </nav>

        <!-- Main Content -->
        <div class="flex flex-col items-center justify-center min-h-[calc(100vh-80px)] px-6">
            <!-- Title -->
            <h1 class="text-6xl font-bold mb-8 text-white">ethereal</h1>

            <!-- Quick Access Buttons -->
            <div class="grid grid-cols-4 gap-4 mb-8 max-w-4xl">
                <!-- Top Row -->
                <button onclick={openAmazon} class="flex items-center space-x-3 px-6 py-4 bg-white rounded-lg hover:bg-gray-100 transition-colors">
                    <div class="w-6 h-6 bg-orange-500 rounded flex items-center justify-center">
                        <span class="text-white font-bold text-xs">A</span>
                    </div>
                    <span class="text-gray-900 font-medium">amazon</span>
                </button>
                
                <button onclick={openChatGPT} class="flex items-center space-x-3 px-6 py-4 bg-teal-500 rounded-lg hover:bg-teal-600 transition-colors">
                    <div class="w-6 h-6 bg-white rounded flex items-center justify-center">
                        <span class="text-teal-500 font-bold text-xs">C</span>
                    </div>
                    <span class="text-white font-medium">ChatGPT</span>
                </button>
                
                <button onclick={openDiscord} class="flex items-center space-x-3 px-6 py-4 bg-blue-600 rounded-lg hover:bg-blue-700 transition-colors">
                    <div class="w-6 h-6 bg-white rounded flex items-center justify-center">
                        <span class="text-blue-600 font-bold text-xs">D</span>
                    </div>
                    <span class="text-white font-medium">Discord</span>
                </button>
                
                <button onclick={openGitHub} class="flex items-center space-x-3 px-6 py-4 bg-gray-800 rounded-lg hover:bg-gray-700 transition-colors">
                    <div class="w-6 h-6 bg-white rounded flex items-center justify-center">
                        <span class="text-gray-800 font-bold text-xs">G</span>
                    </div>
                    <span class="text-white font-medium">GitHub</span>
                </button>

                <!-- Bottom Row -->
                <button onclick={openGoogle} class="flex items-center space-x-3 px-6 py-4 bg-white rounded-lg hover:bg-gray-100 transition-colors">
                    <div class="w-6 h-6 bg-gradient-to-r from-blue-500 via-red-500 to-yellow-500 rounded flex items-center justify-center">
                        <span class="text-white font-bold text-xs">G</span>
                    </div>
                    <span class="text-gray-900 font-medium">Google</span>
                </button>
                
                <button onclick={openPinterest} class="flex items-center space-x-3 px-6 py-4 bg-red-600 rounded-lg hover:bg-red-700 transition-colors">
                    <div class="w-6 h-6 bg-white rounded flex items-center justify-center">
                        <span class="text-red-600 font-bold text-xs">P</span>
                    </div>
                    <span class="text-white font-medium">Pinterest</span>
                </button>
                
                <button onclick={openTwitch} class="flex items-center space-x-3 px-6 py-4 bg-purple-600 rounded-lg hover:bg-purple-700 transition-colors">
                    <div class="w-6 h-6 bg-white rounded flex items-center justify-center">
                        <span class="text-purple-600 font-bold text-xs">T</span>
                    </div>
                    <span class="text-white font-medium">twitch</span>
                </button>
                
                <button onclick={openYouTube} class="flex items-center space-x-3 px-6 py-4 bg-red-600 rounded-lg hover:bg-red-700 transition-colors">
                    <div class="w-6 h-6 bg-white rounded flex items-center justify-center">
                        <span class="text-red-600 font-bold text-xs">Y</span>
                    </div>
                    <span class="text-white font-medium">YouTube</span>
                </button>
            </div>

            <!-- Status Information -->
            <div class="flex items-center space-x-6 text-white">
                <div class="flex items-center space-x-2">
                    <Sun class="w-4 h-4 text-yellow-400" />
                    <span>Good morning, User!</span>
                </div>
                <div class="flex items-center space-x-2">
                    <div class="w-2 h-2 bg-yellow-400 rounded-full"></div>
                    <span>Ping: 148 ms</span>
                </div>
            </div>
        </div>

        <!-- Footer -->
        <div class="absolute bottom-4 left-1/2 transform -translate-x-1/2 text-center text-gray-500 text-sm">
            GitHub • Discord
        </div>
    </div>
{/if}
