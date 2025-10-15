const CACHE_NAME = 'creditsbook-netlify-v2';
const urlsToCache = [
  '/',
  '/index.html',
  '/manifest.json'
];

// Install event - cache essential files
self.addEventListener('install', event => {
  console.log('🛠 Service Worker installing on Netlify...');
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('📦 Opened cache and adding files:', urlsToCache);
        return cache.addAll(urlsToCache)
          .then(() => {
            console.log('✅ All files cached successfully');
          })
          .catch(error => {
            console.log('❌ Cache addAll failed:', error);
          });
      })
  );
  // Force the waiting service worker to become active
  self.skipWaiting();
});

// Activate event - clean up old caches
self.addEventListener('activate', event => {
  console.log('🎯 Service Worker activating...');
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== CACHE_NAME) {
            console.log('🗑 Deleting old cache:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
  // Take control immediately
  self.clients.claim();
});

// Fetch event - serve from cache first, then network
self.addEventListener('fetch', event => {
  // Skip non-GET requests
  if (event.request.method !== 'GET') {
    return;
  }

  // Skip Chrome extensions
  if (event.request.url.startsWith('chrome-extension://')) {
    return;
  }

  // Skip external URLs (Google Apps Script, etc.)
  if (!event.request.url.startsWith(self.location.origin)) {
    return;
  }

  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // Return cached version if found
        if (response) {
          console.log('📂 Serving from cache:', event.request.url);
          return response;
        }

        // Otherwise fetch from network
        console.log('🌐 Fetching from network:', event.request.url);
        return fetch(event.request)
          .then(response => {
            // Check if we received a valid response
            if (!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }

            // Clone the response
            const responseToCache = response.clone();

            // Add to cache for future visits
            caches.open(CACHE_NAME)
              .then(cache => {
                cache.put(event.request, responseToCache);
                console.log('💾 Cached new resource:', event.request.url);
              });

            return response;
          })
          .catch(error => {
            console.log('❌ Fetch failed:', error);
            // You could return a custom offline page here
            throw error;
          });
      })
  );
});

// Handle messages from the client
self.addEventListener('message', event => {
  if (event.data && event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});
