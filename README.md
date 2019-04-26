# PWA Progressive Web Apps - Perusteet

Alla esimerkissä on käytetty editorina VS Code:a, johon on asennettuna [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) lisäosa. IDE-valinnalla ei käytännössä ole merkitystä PWA järjestelmän toteutukseeen. 

## Uuden PWA web-sovelluksen toteutus vaiheittain

- Lisää tyhjä kansio ja avaa VS Code tässä kansiossa, tai käynnistä VC Code ja valitse kyseinen kansio.
- Lisää projektiin tyhjä `index.html`.
- Lisää projektiin tyhjä `styles.css`.
- Lisää projektiin tyhjä `manifest.json` ja lisää sen sisällöksi esim.
```
{
  "name": "PWA Sample Project",
  "short_name": "PWA-Sample",
  "description": "This is my PWA Sample Project",
  "Scope": "/",
  "start_url": "/",
  "splash_pages": null,
  "theme_color": "#2196f3",
  "background_color": "#2196f3",
  "display": "standalone",
  "icons": [
    {
      "src": "/images/logo-192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "/images/logo-512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ]
}  
```

- `manifest.json` tiedoston sekä eri kokoiset ikonit voi generoida esim. tällä generaattorilla https://app-manifest.firebaseapp.com/ 
  - Vain 192x192px ja 512x512px kokoiset ikonit ovat pakollisia.
- Lisätietoja `manifest.json` tiedoston parametreistä ym. löytyy täältä https://developers.google.com/web/fundamentals/web-app-manifest/
- Lisää viittaus `manifest.json` tiedostoon `index.html`:n.
```link rel="manifest" href="/manifest.json">```
- Lisää projektiin uusi javascript-tiedosto `sw.js`.
- Lisää `sw.js` sisällöksi:

```
// Sample to cache files for off-line use

var cacheName = 'PWA-SAMPLE-CACHE';

// Add below files to cache
var filesToCache = [
    '/',                // index.html
    '/sw.js',
    '/styles.css'
];

self.addEventListener('install', function (event) {
    event.waitUntil(
        caches.open(cacheName)
        .then(function (cache) {
            console.info('[sw.js] cached all files');
            return cache.addAll(filesToCache);
        })
    );
});

self.addEventListener('fetch', function (event) {
    event.respondWith(
        caches.match(event.request)
        .then(function (response) {
            if (response) {
                return response
            }
            else {
                // clone request stream
                // as stream once consumed, can not be used again
                var reqCopy = event.request.clone();
                return fetch(reqCopy, { credentials: 'include' }) // reqCopy stream consumed
                .then(function (response) {
                    // bad response
                    // response.type !== 'basic' means third party origin request
                    if (!response || response.status !== 200 || response.type !== 'basic') {
                        return response; // response stream consumed
                    }
                    // clone response stream
                    // as stream once consumed, can not be used again
                    var resCopy = response.clone();
                    // ================== IN BACKGROUND ===================== //
                    // add response to cache and return response
                    caches.open(cacheName)
                    .then(function (cache) {
                        return cache.put(reqCopy, resCopy); // reqCopy, resCopy streams consumed
                    });
                    return response; // response stream consumed
                })
            }
        })
    );
});
```
- Yllä kuvattu  `sw.js` cachettaa halutut tiedostot offline käyttöä varten. Cachetettava tiedostot määritetään `sw.js` tiedoston alussa olevalla listalla.
- Lisätietoja Service Worker avulla toteutettavista toiminnoista ja parametreista löytyy täältä https://serviceworke.rs/
- Lisää `index.html` vielä Service Worker:in kutsu, jonka jälkeen `index.html` näyttää tältä
```
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" />
	<meta charset="utf-8" />
    <link rel="manifest" href="/manifest.json">
    <link rel="icon" type="image/png" href="/images/favicon.ico">
    <link href="styles.css" rel="stylesheet" />
    <meta name="theme-color" content="#317EFB" />
</head>
<body>
    <h1>PWA Sample page 1</h1>
    <script>
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker
                .register('/sw.js')
                .then(function () { console.log('Service Worker Registered'); });
        }
    </script>
</body>
</html>
```

Yllä kuvattu koodi löytyy kokonaisuudessaan täältä: https://github.com/nsdnwe/Pwa-Sample

## Live Server

Mikäli Live Server lisäosaa ei ole asennettuna VS Code:n ja haluat käyttää sitä kehityksessä web-palvelimena.
- Valitse VS Coden Extension sivulta Live Server ja valitse Install.
<img src="https://nsdwww.azurewebsites.net/github-images/image013.png" width="30%">
- Käynnistä Live Server, Go Live painikkeesta.
<img src="https://nsdwww.azurewebsites.net/github-images/image015.png" width="30%">

## Testaus Chromessa

- Avaa Chrome Dev Tools eli paina F12.
- Avaa Applications-välilehti. Mikäli valinta ei ole näkyvillä, se löytyy >> valinnan alta.
<img src="https://nsdwww.azurewebsites.net/github-images/image001.png" width="60%">
- Manifest välilehti näyttää perustiedot ja virheilmoituksia, mikäli jokin ei ole kunnossa.
- Service Workers välilehti näyttää listan mahdollisista virheistä.
<img src="https://nsdwww.azurewebsites.net/github-images/image003.png" width="60%">
- Update on reload rasti kannattaa laittaa päälle.
- Offline tilaa voi testata vaihtamalla Offline ja Update on reload rastit.
<img src="https://nsdwww.azurewebsites.net/github-images/image005.png" width="60%">
 
## Muutosten päivittyminen

Mikäli tuntuu siltä, että `index.html` ym. tiedostoihin muutokset eivät päivity web-palvelimen tuottamille sivulle:
- Valitse Clear storage välilehti ja paina Clear site data painiketta
<img src="https://nsdwww.azurewebsites.net/github-images/image007.png" width="60%">
 
## Lighthouse-testaus

PWA testauksessa suosittelen käyttämään `Google Lighthouse` Chrome lisäosaa, jonka voi asentaa [täältä](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=en).

- Avaa PWA sivu selaimessa ja käynnistä Lighouse.
- Valitse Generate report.
<img src="https://nsdwww.azurewebsites.net/github-images/image009.png" width="40%">
 
- Avaa raportin Progressive Web App osio.
<img src="https://nsdwww.azurewebsites.net/github-images/image011.png" width="60%">
 
- Jotta HTTP => HTTPS redirect virheilmoitusta ei tule, täytyy tuotantoympäristössä eli esim. Azure Web App määrityksistä määrittää HTTPS redirect pakolliseksi
- Virheilmoitusten oikeassa yläkulmassa olevasta nuolesta löytyy suositus, millä virheen voi korjata sekä lisätietoja Learn more linkistä.
