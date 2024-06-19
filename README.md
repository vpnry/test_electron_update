# Gui


To add an auto-update feature to your Electron app using electron-builder and electron-updater, follow these steps:

Step 1: Set Up GitHub Repository
Ensure your app's GitHub repository has releases enabled. You will use these releases to host your update files.

Step 2: Install Required Packages
Ensure you have the necessary packages installed:

bash
Copy
npm install electron-updater
Step 3: Configure electron-builder for Auto-Updates
In your package.json, add the publish section under build:

json
Copy
"build": {
  "appId": "dhamma.tipitakapali.org",
  "productName": "Tipitakapali.org_Offline",
  "icon": "build/icons/win/icon.ico",
  "publish": [
    {
      "provider": "github",
      "owner": "your-github-username",
      "repo": "your-repo-name"
    }
  ],
  ...
}
Replace your-github-username and your-repo-name with your actual GitHub username and repository name.

Step 4: Implement Auto-Update Logic in Your Main Process
In your mainElectron.min.js, add the following code to handle auto-updates:

javascript
Copy
const { app, BrowserWindow, dialog, ipcMain } = require('electron');
const { autoUpdater } = require('electron-updater');

let mainWindow;

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false,
    },
  });

  mainWindow.loadFile('index.html');

  // Check for updates after the window is created
  mainWindow.once('ready-to-show', () => {
    autoUpdater.checkForUpdatesAndNotify();
  });
}

app.on('ready', createWindow);

// Auto Updater events
autoUpdater.on('update-available', () => {
  dialog.showMessageBox({
    type: 'info',
    title: 'Update Available',
    message: 'A new version is available. Do you want to download it now?',
    buttons: ['Yes', 'Later']
  }).then(result => {
    if (result.response === 0) { // 'Yes' button
      autoUpdater.downloadUpdate();
    }
  });
});

autoUpdater.on('update-downloaded', () => {
  dialog.showMessageBox({
    type: 'info',
    title: 'Update Ready',
    message: 'Update downloaded. Do you want to install it now?',
    buttons: ['Yes', 'Later']
  }).then(result => {
    if (result.response === 0) { // 'Yes' button
      autoUpdater.quitAndInstall();
    } else {
      ipcMain.on('quit-and-install', () => {
        autoUpdater.quitAndInstall();
      });
    }
  });
});
Step 5: Handle Update Installation on User Command
In your index.html or renderer process, add a button for users to install the update later if they choose not to install immediately:

html
Copy
<!DOCTYPE html>
<html>
<head>
  <title>TipitakaPali</title>
</head>
<body>
  <h1>Welcome to TipitakaPali Offline!</h1>
  <button id="installUpdate">Install Update</button>

  <script>
    const { ipcRenderer } = require('electron');

    document.getElementById('installUpdate').addEventListener('click', () => {
      ipcRenderer.send('quit-and-install');
    });
  </script>
</body>
</html>
Step 6: Build and Publish Your App
Build your app with electron-builder:

bash
Copy
npm run build-win
After building your app, go to the dist folder and upload the generated .exe and latest.yml files to your GitHub releases.

Step 7: Test Your Auto-Update Feature
Ensure you have an existing version of your app installed.
Upload a new release to GitHub with a higher version number.
Run the app and check if it notifies you about the new update and handles the download and installation process correctly.
By following these steps, you will have added auto-update functionality to your Electron app, allowing it to check for updates, notify users, and download and install updates as needed.


