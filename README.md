# Throttle Riot

An arcade motorcycle combat racer in the spirit of the 90s classics: race eight rivals down four
roads, punch and kick them off their bikes, steal their weapons, dodge traffic, and don't get
caught by the cops. It runs in any modern browser, installs on phones as an app, and can be
built into an Android APK.

Everything (graphics, sound, music) is generated in code. There are no image or audio files
besides the app icons, so the whole game is one `index.html`.

## What's in the repo

| File | Purpose |
|---|---|
| `index.html` | The entire game |
| `manifest.json` | Makes it installable as an app (name, icon, landscape, fullscreen) |
| `sw.js` | Service worker, so the game works offline after the first visit |
| `icons/` | App icons for Android, iOS and the browser tab |
| `.nojekyll` | Tells GitHub Pages to serve files as-is |
| `package.json`, `capacitor.config.json` | Settings for wrapping the game as a native Android app |
| `.github/workflows/android-apk.yml` | Builds an APK for you on GitHub, no Android Studio needed |
| `assets/icon-only.png` | High-res icon used when generating the Android app icon |

## 1. Put it on GitHub Pages (web app)

1. Sign in at github.com and click **New repository**. Name it, for example, `throttle-riot`.
   Make it **Public** (GitHub Pages is free for public repos). Click **Create repository**.
2. On the new repo page click **uploading an existing file**. Drag in everything from this folder,
   including the `icons`, `assets` and `.github` folders. Click **Commit changes**.
   - Hidden items like `.github` and `.nojekyll` may not show in your file browser. On Mac press
     `Cmd+Shift+.` in Finder; on Windows enable *View → Hidden items*. Or use Git (below).
3. Go to **Settings → Pages**. Under *Build and deployment* set **Source: Deploy from a branch**,
   **Branch: main**, folder **/ (root)**, then **Save**.
4. Wait a minute or two and refresh. The page shows your live address:
   `https://YOUR-USERNAME.github.io/throttle-riot/`

Using Git from a terminal instead:

```bash
cd throttle-riot
git init
git add .
git commit -m "Throttle Riot"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/throttle-riot.git
git push -u origin main
```

**Updating the game later:** edit `index.html`, then open `sw.js` and change
`throttle-riot-v1` to `throttle-riot-v2` (and so on) so phones pick up the new version. Commit and push.

## 2. Install on a phone as an app (no store needed)

This is the quickest way and works on both platforms. The game runs fullscreen from a home-screen
icon and works offline after the first launch.

**Android (Chrome):** open your GitHub Pages address, tap the **Install app** button on the game's
menu (or Chrome's **⋮ menu → Add to Home screen / Install app**). The icon appears in your app drawer.

**iPhone / iPad (Safari):** open the address in **Safari** (not Chrome), tap the **Share** button,
scroll and tap **Add to Home Screen**, then **Add**. Launch it from the new icon.
If the menu is silent, check the phone's ring/silent switch; iOS mutes web audio in silent mode.

## 3. Get a real Android APK

### Option A: let GitHub build it (recommended)

The included workflow wraps the game with Capacitor and builds an APK in the cloud.

1. After uploading the files, open the **Actions** tab of your repo. If asked, click
   **I understand my workflows, go ahead and enable them**.
2. Click **Build Android APK** on the left, then **Run workflow → Run workflow**.
   It also runs automatically whenever you push a change to `index.html`.
3. Wait for the green check (about 5–8 minutes). Open the run and download
   **ThrottleRiot-apk** under *Artifacts*. Unzip it to get `app-debug.apk`.
4. Copy `app-debug.apk` to your phone (USB, Google Drive, email, etc.) and tap it.
   Android will ask you to allow installs from that source (*Settings → Install unknown apps*).
   Allow it, then tap **Install**.

This is a debug-signed APK, which is fine for installing on your own devices and sharing with friends.
To publish on Google Play you need a release build signed with your own key (Android Studio:
*Build → Generate Signed App Bundle / APK*), and a Google Play developer account.

### Option B: PWABuilder (no code, uses your live site)

1. Go to **pwabuilder.com**, paste your GitHub Pages address, click **Start**.
2. Click **Package for stores → Android → Generate package**, and download the zip.
3. Inside you'll find an `.apk` to install directly and an `.aab` for Google Play,
   plus a signing key. Keep the key safe; you need it for every future update.

### Option C: build on your own computer

Requires Node.js 22+, JDK 21 and Android Studio.

```bash
npm install
npm run android:add      # copies the game to www/ and creates the android/ project
npx cap open android     # opens Android Studio
```

In Android Studio choose **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
After changing `index.html`, run `npm run android:sync` and build again.

## 4. iOS native app (optional)

Apple doesn't allow installing an app file the way Android does, so the home-screen install in
step 2 is the practical route for iPhone. If you want a true native app, you need a Mac with Xcode:

```bash
npm install
npm install @capacitor/ios
npm run www
npx cap add ios
npx cap open ios
```

In Xcode, pick your Apple ID under *Signing & Capabilities* and press Run with your iPhone connected.
With a free Apple ID the app expires after 7 days; the App Store requires the paid Apple Developer Program.

## Controls

**Touch:** slide your thumb across the left pad to steer. Punch (hold to keep swinging), Kick,
Nitro and Brake are on the right. Throttle is automatic by default; turn it off in Settings to get a
Gas button. Tilt steering is available in Settings too.

**Keyboard:** arrows or WASD to steer, accelerate and brake. Z or J to punch, X or K to kick,
Space for nitro, P or Esc to pause.

## How the game works

- **Races:** four roads (desert, coast, forest, neon city), each unlocked by finishing top 3 on the previous one.
- **Combat:** knocking a rival down pays cash and refills nitro. Knock down someone holding a bat, chain
  or pipe and you take it. Kicks shove rivals off the road or into traffic.
- **Damage:** your rider has stamina and your bike has condition. Every crash costs bike condition;
  at zero the bike is wrecked and the race ends. Crash with the police bike nearby and you're busted and fined.
- **Garage:** buy the Viper 600 and Tempest 1000, and upgrade engine, tyres, armor and nitro.
- Progress is saved on the device automatically.

## Tuning it

Handy constants near the top of the script in `index.html`:

- `TRACKS` — each road's length, curviness, hills, traffic, rival speed (`opp`) and payout (`mult`)
- `RIVALS` — names, colours, weapons, aggression and skill of each rider
- `BIKES` and `UPGRADES` — prices and performance
- `PRIZES` — prize money by finishing place

The renderer is a classic "pseudo-3D" road engine: the road is thousands of short segments
projected to the screen each frame, with sprites scaled by distance. Graphics quality adapts
automatically on slower phones (or set it manually in Settings).
