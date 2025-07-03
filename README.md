// ==UserScript==
// @name         Rainbet Currency Patch (FUN → RUB)
// @namespace    http://tampermonkey.net/
// @version      1.2
// @description  Replaces FUN currency with RUB in BGaming Aviamasters init responses only
// @match        https://demo.rainbet-network.com/games/Aviamasters/*
// @grant        none
// @run-at       document-start
// @noframes
// ==/UserScript==

(function () {
    'use strict';

  
    const originalFetch = window.fetch;

    window.fetch = async (...args) => {
        const [url, config] = args;

   
        const isTarget = url.includes("/api/Aviamasters/") &&
                         config?.method?.toUpperCase() === "POST" &&
                         config?.body?.includes('"command":"init"');

        if (isTarget) {
            console.log("[TM] Intercepting init request:", url);

            const response = await originalFetch(...args);
            const cloned = response.clone();
            const json = await cloned.json();

            if (
                json?.options?.currency?.code === "FUN" &&
                json?.flow?.command === "init"
            ) {
                console.log("[TM] Patching currency FUN → RUB");

 
                json.options.currency.code = "RUB";
                json.options.currency.symbol = "₽";

 
                return new Response(JSON.stringify(json), {
                    status: response.status,
                    statusText: response.statusText,
                    headers: response.headers,
                });
            }

            return response;
        }

        return originalFetch(...args);
    };
})();
