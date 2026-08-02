// Service Worker - EURL DJELONG PAPIERS
// Caches the app shell so the app can still open (offline) once it has been
// visited at least once. Firebase data itself still needs an internet
// connection to sync in real time.

const CACHE_NAME = 'djelong-app-v3-1-0';
const APP_SHELL = [
    './',
    './index.html'
];

self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then((cache) => cache.addAll(APP_SHELL).catch(() => {}))
            .then(() => self.skipWaiting())
    );
});

self.addEventListener('activate', (event) => {
    event.waitUntil(
        caches.keys().then((keys) =>
            Promise.all(keys.filter((k) => k !== CACHE_NAME).map((k) => caches.delete(k)))
        ).then(() => self.clients.claim())
    );
});

self.addEventListener('fetch', (event) => {
    // Network-first for Firebase / API calls, cache-first for everything else.
    const url = event.request.url;
    if (url.includes('firebaseio.com') || url.includes('googleapis.com')) {
        return; // let these go straight to the network
    }
    event.respondWith(
        caches.match(event.request).then((cached) => {
            const networkFetch = fetch(event.request)
                .then((response) => {
                    if (response && response.status === 200) {
                        const clone = response.clone();
                        caches.open(CACHE_NAME).then((cache) => cache.put(event.request, clone));
                    }
                    return response;
                })
                .catch(() => cached);
            return cached || networkFetch;
        })
    );
});
