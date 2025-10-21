# **Plan: Building a Tauri-Based SearXNG Web Browser**

This document provides a highly detailed, step-by-step procedure for creating a simple, privacy-focused web browser using the Tauri framework. The browser will use SearXNG as its default search engine.

## **Part 1: Prerequisites & Initial Setup**

Before writing any code, we must ensure the development environment is ready and create the initial project structure.

### **Step 1.1: Install System Dependencies**

You need Rust, Node.js, and specific libraries for your operating system.

1. **Install Rust:** If you don't have it, install Rust and its package manager, Cargo, via rustup. Follow the instructions at [rustup.rs](https://rustup.rs/).  
2. **Install Node.js:** Tauri uses Node.js to manage the webview frontend. Install it from the [official Node.js website](https://nodejs.org/).  
3. **Install Tauri Prerequisites:** Follow the official Tauri guide for your specific operating system to install necessary build tools like C++ compilers and webview libraries.  
   * **Windows:** [Tauri Windows Prerequisites](https://www.google.com/search?q=https://tauri.app/v1/guides/getting-started/prerequisites%23windows)  
   * **macOS:** [Tauri macOS Prerequisites](https://www.google.com/search?q=https://tauri.app/v1/guides/getting-started/prerequisites%23macos)  
   * **Linux:** [Tauri Linux Prerequisites](https://www.google.com/search?q=https://tauri.app/v1/guides/getting-started/prerequisites%23linux)  
4. **Install Tauri CLI:** This command-line tool is essential for managing Tauri projects. Install it globally using Cargo:  
   cargo install tauri-cli

### **Step 1.2: Create the Tauri Project**

Now, we will initialize the project.

1. Navigate to the directory where you want to create your project.  
2. Run the following command in your terminal:  
   cargo tauri init

3. You will be prompted with several questions. Answer them as follows:  
   * **What is your app name?** searxng-browser  
   * **What should the window title be?** SearXNG Browser  
   * **What UI recipe would you like to add?** html (Select this by navigating with arrow keys and pressing Enter)  
   * **Where are your web assets (HTML, CSS, JS) located, relative to the "\<current dir\>/src-tauri" folder?** ../  
   * **What is the url of your dev server?** ../  
   * **What is your frontend dev command?** Leave this blank and press Enter.  
   * **What is your frontend build command?** Leave this blank and press Enter.

This will create a new directory named searxng-browser with the default Tauri file structure.

## **Part 2: Configuring the Project Manifests**

Next, we update the configuration files to define our application's metadata, dependencies, and permissions.

### **Step 2.1: Update Rust Dependencies (Cargo.toml)**

Open the searxng-browser/Cargo.toml file. This file manages our Rust dependencies. Replace its entire content with the following to include Tauri and serialization libraries:

\[package\]  
name \= "searxng-browser"  
version \= "0.1.0"  
description \= "A simple Tauri-based web browser using SearXNG"  
authors \= \["you"\]  
license \= ""  
repository \= ""  
edition \= "2021"

\# See more keys and their definitions at \[https://doc.rust-lang.org/cargo/reference/manifest.html\](https://doc.rust-lang.org/cargo/reference/manifest.html)

\[build-dependencies\]  
tauri-build \= { version \= "1.5", features \= \[\] }

\[dependencies\]  
tauri \= { version \= "1.6", features \= \[ "shell-open", "window-all", "webview-find"\] }  
serde \= { version \= "1.0", features \= \["derive"\] }  
serde\_json \= "1.0"

\[features\]  
\# this feature is used for production builds or when \`devPath\` points to the filesystem  
\# DO NOT REMOVE\!\!  
custom-protocol \= \["tauri/custom-protocol"\]

### **Step 2.2: Update Tauri Configuration (tauri.conf.json)**

Open the searxng-browser/src-tauri/tauri.conf.json file. This file controls window behavior, permissions, and bundle information. Replace its entire content with the configuration below:

{  
  "build": {  
    "beforeDevCommand": "",  
    "beforeBuildCommand": "",  
    "devPath": "../",  
    "distDir": "../",  
    "withGlobalTauri": true  
  },  
  "package": {  
    "productName": "SearXNG Browser",  
    "version": "0.1.0"  
  },  
  "tauri": {  
    "allowlist": {  
      "all": false,  
      "window": {  
        "all": true  
      },  
      "shell": {  
        "all": false,  
        "open": true  
      }  
    },  
    "bundle": {  
      "active": true,  
      "targets": "all",  
      "identifier": "com.tauri.searxng.browser",  
      "icon": \[  
        "icons/32x32.png",  
        "icons/128x128.png",  
        "icons/128x128@2x.png",  
        "icons/icon.icns",  
        "icons/icon.ico"  
      \]  
    },  
    "security": {  
      "csp": null  
    },  
    "windows": \[  
      {  
        "fullscreen": false,  
        "resizable": true,  
        "title": "SearXNG Browser",  
        "width": 1024,  
        "height": 768,  
        "label": "main"  
      }  
    \]  
  }  
}

## **Part 3: Building the Browser's Core**

Now we will write the code for the frontend UI and the backend logic.

### **Step 3.1: Create the Frontend User Interface (index.html)**

Open the searxng-browser/index.html file. This file defines the visual layout of our browser. Replace its entire content with the following HTML, CSS, and JavaScript code:

\<\!DOCTYPE html\>  
\<html lang="en"\>  
\<head\>  
    \<meta charset="UTF-8" /\>  
    \<meta name="viewport" content="width=device-width, initial-scale=1.0" /\>  
    \<title\>SearXNG Browser\</title\>  
    \<script src="\[https://cdn.tailwindcss.com\](https://cdn.tailwindcss.com)"\>\</script\>  
    \<style\>  
        body, html {  
            margin: 0;  
            padding: 0;  
            height: 100vh;  
            overflow: hidden;  
            font-family: 'Inter', sans-serif;  
            background-color: \#1a1a1a;  
        }  
        \#controls {  
            backdrop-filter: blur(10px);  
        }  
        \#url-input:focus {  
            box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.5);  
        }  
    \</style\>  
\</head\>  
\<body class="flex flex-col h-screen"\>

    \<\!-- Navigation Controls \--\>  
    \<div id="controls" class="flex items-center p-2 bg-gray-800 bg-opacity-80 border-b border-gray-700 shadow-md"\>  
        \<button id="back-btn" class="p-2 rounded-full hover:bg-gray-700 transition-colors focus:outline-none"\>  
            \<svg xmlns="\[http://www.w3.org/2000/svg\](http://www.w3.org/2000/svg)" class="h-5 w-5 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"\>  
                \<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" /\>  
            \</svg\>  
        \</button\>  
        \<button id="forward-btn" class="p-2 rounded-full hover:bg-gray-700 transition-colors focus:outline-none ml-1"\>  
            \<svg xmlns="\[http://www.w3.org/2000/svg\](http://www.w3.org/2000/svg)" class="h-5 w-5 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor"\>  
                \<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" /\>  
            \</svg\>  
        \</button\>  
        \<div class="relative flex-grow mx-2"\>  
            \<input type="text" id="url-input" class="w-full bg-gray-900 text-white rounded-full px-4 py-2 text-sm focus:outline-none border-2 border-transparent focus:border-blue-500 transition-colors" placeholder="Search with SearXNG or enter a URL"\>  
        \</div\>  
    \</div\>

    \<\!-- Browser View \--\>  
    \<iframe id="browser-view" src="\[https://searx.be/\](https://searx.be/)" class="flex-grow w-full h-full border-none"\>\</iframe\>

    \<script\>  
        const { invoke } \= window.\_\_TAURI\_\_.tauri;

        const urlInput \= document.getElementById('url-input');  
        const backBtn \= document.getElementById('back-btn');  
        const forwardBtn \= document.getElementById('forward-btn');  
        const browserView \= document.getElementById('browser-view');

        urlInput.addEventListener('keydown', (event) \=\> {  
            if (event.key \=== 'Enter') {  
                const value \= urlInput.value;  
                invoke('go\_to\_url', { url: value });  
            }  
        });  
          
        backBtn.addEventListener('click', () \=\> invoke('go\_back'));  
        forwardBtn.addEventListener('click', () \=\> invoke('go\_forward'));

        browserView.addEventListener('load', () \=\> {  
            try {  
                const newUrl \= browserView.contentWindow.location.href;  
                if (\!newUrl.startsWith('about:blank')) {  
                    urlInput.value \= newUrl;  
                }  
                const title \= browserView.contentDocument.title;  
                invoke('update\_title', { title: title });  
            } catch (e) {  
                // Cannot access iframe content from a different origin  
                console.warn("Cross-origin frame error. Can't update URL bar from iframe.", e);  
            }  
        });  
    \</script\>  
\</body\>  
\</html\>

### **Step 3.2: Implement the Backend Logic (main.rs)**

Open the searxng-browser/src-tauri/src/main.rs file. This Rust file is the heart of the application, handling the browser's core logic. Replace its entire content with the following Rust code:

// Prevents additional console window on Windows in release, DO NOT REMOVE\!\!  
\#\!\[cfg\_attr(not(debug\_assertions), windows\_subsystem \= "windows")\]

use tauri::{command, Manager, Window};

\#\[command\]  
fn go\_back(window: Window) {  
    let \_ \= window.eval("document.getElementById('browser-view').contentWindow.history.back();");  
}

\#\[command\]  
fn go\_forward(window: Window) {  
    let \_ \= window.eval("document.getElementById('browser-view').contentWindow.history.forward();");  
}

\#\[command\]  
fn go\_to\_url(window: Window, url: String) {  
    let mut final\_url \= url;  
    if \!final\_url.starts\_with("http://") && \!final\_url.starts\_with("https://") {  
        // Default to a SearXNG instance for search queries  
        final\_url \= format\!("\[https://searx.be/search?q=\](https://searx.be/search?q=){}", final\_url);  
    }  
    let script \= format\!("document.getElementById('browser-view').src \= '{}';", final\_url);  
    let \_ \= window.eval(\&script);  
}

\#\[command\]  
fn update\_title(window: Window, title: String) {  
    let \_ \= window.set\_title(\&title);  
}

fn main() {  
    tauri::Builder::default()  
        .setup(|app| {  
            let window \= app.get\_window("main").unwrap();  
            let window\_clone \= window.clone();  
            let iframe \= window.find\_webview("browser-view").unwrap();  
              
            iframe.on\_page\_load(move |\_, payload| {  
                println\!("Page loaded: {:?}", payload.url());  
                let get\_title\_script \= "document.title";  
                 let window\_clone2 \= window\_clone.clone();  
                let \_ \= window\_clone.eval\_script(get\_title\_script).map(move |title| {  
                    if let Ok(title\_str) \= title.as\_str().map(String::from) {  
                         let \_ \= window\_clone2.set\_title(\&title\_str);  
                    }  
                });  
            });  
            Ok(())  
        })  
        .invoke\_handler(tauri::generate\_handler\!\[  
            go\_back,  
            go\_forward,  
            go\_to\_url,  
            update\_title  
        \])  
        .run(tauri::generate\_context\!())  
        .expect("error while running tauri application");  
}

## **Part 4: Running and Building the Application**

With all the code and configuration in place, you can now run your new browser.

### **Step 4.1: Run in Development Mode**

1. Open your terminal in the root of the searxng-browser directory.  
2. Run the following command:  
   cargo tauri dev

3. Cargo will compile the Rust backend, and then a window for your browser application will appear. You can now use the address bar to search or navigate to websites.

### **Step 4.2: Build for Production**

Once you are happy with the browser, you can create a distributable, standalone application.

1. In your terminal, run the build command:  
   cargo tauri build

2. Tauri will build and bundle your application into a native format for your operating system (e.g., .exe on Windows, .app on macOS, .AppImage or .deb on Linux).  
3. You can find the final executable file in the searxng-browser/src-tauri/target/release/bundle/ directory.

This plan covers the complete process from initialization to a final, runnable application. You can now extend this base with new features like bookmarks, tabs, or history.