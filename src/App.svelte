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
        X
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

    function acceptDisclaimer() {
        disclaimerAccepted = true;
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
                        in their jurisdiction. If you are a school admin and want this taken down contact us at ethereal.proxy1@gmail.com.
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
                        <li>• Do not use for illegal activities we will not be accountable</li>
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
{:else if proxyManager.isProxyOpen}
    <div>
        <iframe
            bind:this={iframe}
            title="Proxy"
            class="w-full h-[91vh] bg-transparent rounded-b-2xl shadow-lg shadow-black/20"
            src={proxyManager.iframeUrl}
            onload={onIframeLoad}
            allow={iframeAllow}
            sandbox={iframeSandbox}
        ></iframe>
    </div>
    <Config bind:isConfigOpen></Config>

    <div
        class="flex grow-1 bottom-0 fixed w-full bg-transparent h-[9%] items-center justify-center"
    >
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-0 mr-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Previous Page"
            onclick={() => setUrl(proxyHistory.goBackward())}
        ><ArrowLeft class="scale-95 transition-all" /></button> 
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-0 mr-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Next Page"
            onclick={() => setUrl(proxyHistory.goForward())}
        ><ArrowRight class="scale-95 transition-all" /></button>    
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-0 mr-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Refresh Page"
            onclick={() => iframe.contentWindow.location.reload()}
        ><RotateCw class="scale-95 transition-all" /></button>
        <input
            type="text"
            class="input max-w-2/3 h-8 min-w-2/3 rounded-full focus:shadow-none focus:border-none focus:brightness-125 transition-all p-5 focus:outline-none outline-none border-none shadow-none"
            title="Destination URL"
            id="searchbar"
            placeholder="loading, please wait. this may take some time."
            bind:value={destinationInput}
            bind:this={searchbar}
            onkeydown={onEnterKeyPressed(startProxy)}
        />
        <span
            class="loading loading-spinner scale-75 loading-xl ml-[-3.5%] mb-[0%] mr-1.5 z-1000 transition-all" style="display: none;" bind:this={loader}>
        </span>
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Start proxy"
            onclick={startProxy}
            disabled={proxyManager.proxyUrl === "" ||
                !proxyManager.serviceWorker}><Search class="scale-95" /></button>
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Open Settings"
            onclick={() => (isConfigOpen = true)}
        ><Settings2 class="scale-95" /></button>
        <button
            class="btn-circle bg-red-500 p-2 text-sm m-0 ml-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="End Proxy"
            onclick={() => (proxyManager.isProxyOpen = false)}><X class="scale-95" /></button>
    </div>
{:else}
    <Config bind:isConfigOpen></Config>

    <div
        class="flex grow-1 bottom-0 fixed w-full bg-transparent h-[9%] items-center justify-center"
    >
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-0 mr-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Previous Page"
            onclick={() => setUrl(proxyHistory.goBackward())}
        ><ArrowLeft class="scale-95 transition-all" /></button> 
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-0 mr-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Next Page"
            onclick={() => setUrl(proxyHistory.goForward())}
        ><ArrowRight class="scale-95 transition-all" /></button>    
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-0 mr-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Refresh Page"
            onclick={() => iframe.contentWindow.location.reload()}
        ><RotateCw class="scale-95 transition-all" /></button>
        <input
            type="text"
            class="input max-w-2/3 h-8 min-w-2/3 rounded-full p-5 focus:outline-none focus:shadow-none focus:border-none focus:brightness-125 transition-all outline-none shadow-none border-none"
            title="Destination URL"
            placeholder="search anything..."
            onkeydown={onEnterKeyPressed(startProxy)}
            bind:this={urlBar}
            bind:value={destinationInput}
        />
        <span
            class="loading loading-spinner scale-75 loading-xl ml-[-3.5%] mb-[0%] mr-1.5 z-1000 transition-all" style="display: none;" bind:this={loader}>
        </span>
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Start proxy"
            onclick={startProxy}
            disabled={proxyManager.proxyUrl === "" ||
                !proxyManager.serviceWorker}><Search class="scale-95" /></button>
        <button
            class="btn-circle bg-blue-500 p-2 text-sm m-0 ml-2 cursor-pointer pointer-events-auto hover:brightness-75 transition-all"
            title="Open Settings"
            onclick={() => (isConfigOpen = true)}
        ><Settings2 class="scale-95" /></button>
    </div>
    <div class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-4/5 items-center text-center">
        <h1 class="text-5xl font-bold mb-2 text-transparent bg-clip-text bg-gradient-to-r from-blue-500 to-purple-300">ethereal</h1>
        <p>---a sleek proxy with speed, design, and usability in mind.---</p>
    </div>
{/if}
