# chromitex
A Framework build around the devtools Protocol that enables to build Cross Platform Desktop Apps using GraalVM


## Qickstarts
Install GraalVM community edition
Install graal-node
Create a Project with npm init
npm install puppeteer (be aware that this installs chromium into the node_modules/puppeteer/.chromium folder you should maybe at present not mess with that as this works on the most platforms at present so if you bundle for multiple platforms run npm install on that platform. 

use the following example code put it into your npm project in index.js
```js
const puppeteer = require('puppeteer-core')
const fs = require('fs');

const loadingPage = `
<title>${encodeURIComponent('')}APP</title>
<style>html{background:${encodeURIComponent('#000000')};}</style>
<script>paramsForReuse = ${JSON.stringify(undefined)};</script>`;

const html5 = `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>${encodeURIComponent('')}APP</title>
    <style>
      html {
        background:${encodeURIComponent('#000000')};
        color: aliceblue;
      }
    </style>
</head>
<body>
    <h1>Loading...</h1>    
</body>
</html>`

//--window-position=0,0 --window-size=1,1
const appMode = {
    executablePath: '/snap/bin/chromium',
    headless: false,
    //ignoreHTTPSErrors: true,
    defaultViewport: null, // Fix window-size
    // userDataDir: './myUserDataDir',
    //width: 800,
    //top: 20,
    ignoreDefaultArgs: [
      '--enable-automation', // No Automation futures
      '--enable-blink-features=IdleDetection' // Disable experimental default flag
    ],
    args: [
      '--enable-features=NetworkService',
      `--app=data:text/html,${html5}`, // Load the App
      '--no-default-browser-check', // Suppress browser check
      //'--window-size=800,800',
      '--start-maximized',
    ]
};

const disableGoogleKeysMissingMessage = () => {
    process.env.GOOGLE_API_KEY = "no";
    process.env.GOOGLE_DEFAULT_CLIENT_ID="no";
    process.env.GOOGLE_DEFAULT_CLIENT_SECRET="no";
}

disableGoogleKeysMissingMessage();

(async () => {
  const browser = await puppeteer.launch(appMode);
  const openPages = await browser.pages();
  openPages.forEach(async (page, i) => {
    if (i === 0) {

      const preloadScript = () => {
        const possibleButtons = [
          /** resetBtn*/ '#fullscreen-wrapper > div.content > div > div:nth-child(2) > div > now-player-box > div > now-player > now-continue-board > div > button.no-break.btn-spacing.button-default.secondary.btnReset.ng-star-inserted',
          /** akzept cookies*/ '#notice > div.message-component.message-row.mobile-reverse > div:nth-child(2) > button',
          /** accept other sttuff */ '#notice > div.message-component.message-row.mobile-reverse > button.message-component.message-button.no-children.focusable.sp_choice_type_11',
        ];

        const observer = new MutationObserver(() => {
          const button = possibleButtons.find((selector) => document.querySelector(selector))
            
          if (button) {
               console.log("It's in the DOM!");
               button.click();
           }
        });
        
        // Registering a Observer that intercepts the "fortsetzen" "restart" and clicks restart
        observer.observe(document, {attributes: false, characterData: false, childList: true, subtree:true});
      }
      
      await page.evaluateOnNewDocument(preloadScript);
      // page.on('response', response => console.log(`${response.status()} ${response.url()}`))
      await page.goto('https://www.tvnow.de/');
      // Save the Login cookie

    } else {
      page.close(); // Close eventual existing popups 
    }
  });
})();
```
