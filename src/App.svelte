<script lang="ts">
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

    // --- Disclaimer state ---
    let showDisclaimer = $state(true);

    // --- Proxy app states ---
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

    function startProxy() {
        if (proxyManager.startProxy(destinationInput)) {
            destinationInput = "";
            console.log("proxy started");
        }
    }

    let iframeHasLoaded = $state(false);
    let isProxyOpen = $state(false);
    let iframe: HTMLIFrameElement = $state();
    let searchbar: HTMLInputElement = $state();

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
        if (!iframe) return;
        const src = iframe.contentWindow.location.pathname;
        if (!src.includes(proxyManager.uvConfig.prefix)) return;

        iframeHasLoaded = true;
                    
        if (searchbar) {
            searchbar.value = proxyManager.url;
            console.log("Set Value: " + proxyManager.url);
        }

        proxyManager.url = proxyManager.uvConfig.decodeUrl(
            src.slice(proxyManager.uvConfig.prefix.length),
        );
    }

    let proxyHistory = new History();
    $effect(() => {
        proxyHistory.addToHistory(proxyManager.url);
    });

    function setUrl(url: string | null) {
        if (url == null) return;
        proxyManager.url = url;
        proxyManager.reloadIframe();
    }

    function acceptDisclaimer() {
        showDisclaimer = false;
    }
</script>

{#if showDisclaimer}
    <!-- DISCLAIMER PAGE -->
    <div class="fixed inset-0 flex flex-col items-center justify-center bg-gray-900 text-white p-8">
        <h1 class="text-4xl font-bold mb-4">Disclaimer</h1>
        <p class="mb-6 text-center max-w-xl">
            By using this proxy, you agree that you are responsible for any actions performed using it. 
            This tool is provided as-is for educational and personal use only.
        </p>
        <button 
            class="px-6 py-3 bg-blue-500 hover:bg-blue-600 rounded text-white text-lg"
            on:click={acceptDisclaimer}>
            I Understand & Continue
        </button>
    </div>
{:else}
    <!-- ORIGINAL APP UI -->
    {#if proxyManager.isProxyOpen}
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
        <!-- your other 'else' UI goes here (search, title, etc.) -->
        <!-- just wrap all your current {:else} code inside this block -->
    {/if}
{/if}
