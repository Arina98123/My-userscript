// ==UserScript==
// @name         Rainbet Currency Patch (FUN → RUB)
// @namespace    http://tampermonkey.net/
// @version      1.3
// @description  Replaces FUN currency with RUB or USD in Rainbet Aviamasters init responses
// @match        https://rainbet.com/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function () {
    'use strict';

    const desiredCurrency = "RUB"; // или "USD"
    const desiredSymbol   = desiredCurrency === "USD" ? "$" : "₽";

    const originalFetch = window.fetch;

    window.fetch = async (...args) => {
        const [url, config] = args;

        const isTarget = url.includes("/api/rainbet/") &&
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
                console.log(`[TM] Patching currency FUN → ${desiredCurrency}`);

                json.options.currency.code = desiredCurrency;
                json.options.currency.symbol = desiredSymbol;

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
