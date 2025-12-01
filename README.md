# 🎥 YouTube Embed Wrapper

A production-ready HTTPS wrapper for safely embedding YouTube videos inside mobile WebViews (Cordova, Capacitor) and normal browsers. This solves the **"Protocols must match"** error on iOS when trying to embed YouTube content in hybrid mobile apps.

## 🚀 Why This Exists

When you embed YouTube videos directly in a Cordova or Capacitor app using an iframe, iOS WebKit often blocks the content with:

```
Refused to display 'https://www.youtube.com/...' in a frame because protocols, domains, or ports must match.
```

**The Solution:** Host a minimal HTTPS wrapper page that embeds the YouTube iframe. Your app iframes the wrapper (both HTTPS), which then embeds YouTube (HTTPS → HTTPS), satisfying the same-origin policy requirements.

## ✨ Features

- ✅ **Fixes iOS WebView protocol mismatch errors**
- ✅ **Supports all YouTube embed parameters** (autoplay, loop, start/end times, etc.)
- ✅ **IFrame Player API integration** with postMessage support
- ✅ **Production-ready security headers** (CSP, Referrer-Policy, etc.)
- ✅ **Zero dependencies** - pure HTML/CSS/JavaScript
- ✅ **Responsive and mobile-optimized**
- ✅ **Graceful error handling** with helpful messages
- ✅ **Privacy-focused** - no analytics by default

## 📦 Quick Start

### 1. Deploy to Netlify

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy)

1. Fork/clone this repository
2. Connect to Netlify (or drag the `public` folder to [drop.netlify.com](https://app.netlify.com/drop))
3. Deploy! Your wrapper will be live at `https://your-site.netlify.app`

### 2. Use in Your Cordova/Capacitor App

In your HTML:

```html
<iframe
  src="https://your-site.netlify.app/?id=dQw4w9WgXcQ&autoplay=1&mute=1"
  style="width:100%; height:220px; border:0;"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; fullscreen"
  allowfullscreen
></iframe>
```

**That's it!** The video will now play correctly on iOS devices.

## 📖 Usage & API

### Basic Embed URL Structure

```
https://your-site.netlify.app/?id={VIDEO_ID}&param1=value1&param2=value2
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `id` | string | **required** | YouTube video ID (11 characters) |
| `autoplay` | 0 or 1 | `0` | Auto-play video on load |
| `mute` | 0 or 1 | `1` | Mute video (required for autoplay on iOS) |
| `controls` | 0 or 1 | `1` | Show player controls |
| `modest` | 0 or 1 | `1` | Modest branding (minimal YouTube logo) |
| `start` | number | - | Start time in seconds |
| `end` | number | - | End time in seconds |
| `loop` | 0 or 1 | `0` | Loop the video |
| `playlist` | string | - | Comma-separated video IDs (required for loop) |

### Examples

**Basic autoplay (muted):**
```
?id=dQw4w9WgXcQ&autoplay=1&mute=1
```

**Start at 30s, end at 90s:**
```
?id=dQw4w9WgXcQ&start=30&end=90
```

**Loop a video:**
```
?id=dQw4w9WgXcQ&loop=1&playlist=dQw4w9WgXcQ
```

**No controls, no branding:**
```
?id=dQw4w9WgXcQ&controls=0&modest=0
```

## 🔌 PostMessage API

The wrapper supports bidirectional communication with the parent window via `postMessage`.

### Send Commands to Player (Parent → Wrapper)

```javascript
// Get the iframe element
const iframe = document.getElementById('youtube-iframe');

// Send command to play
iframe.contentWindow.postMessage({
  type: 'yt-command',
  action: 'playVideo'
}, '*');

// Send command to pause
iframe.contentWindow.postMessage({
  type: 'yt-command',
  action: 'pauseVideo'
}, '*');

// Seek to 30 seconds
iframe.contentWindow.postMessage({
  type: 'yt-command',
  action: 'seekTo',
  args: { value: 30 }
}, '*');
```

### Receive Events from Player (Wrapper → Parent)

```javascript
// Listen for YouTube events
window.addEventListener('message', function(event) {
  if (event.data && event.data.type === 'yt-event') {
    console.log('YouTube event:', event.data.event);
    console.log('Event data:', event.data.info);
    
    // Handle specific events
    if (event.data.event === 'onReady') {
      console.log('Player is ready!');
    } else if (event.data.event === 'onStateChange') {
      console.log('Player state:', event.data.info);
      // -1: unstarted, 0: ended, 1: playing, 2: paused, 3: buffering, 5: cued
    }
  }
});
```

### Available Commands

- `playVideo` - Play the video
- `pauseVideo` - Pause the video
- `stopVideo` - Stop the video
- `seekTo` - Seek to specific time (pass `args: { value: seconds }`)
- `mute` - Mute the video
- `unMute` - Unmute the video
- `setVolume` - Set volume 0-100 (pass `args: { value: volume }`)

### Available Events

- `onReady` - Player is ready
- `onStateChange` - Playback state changed
- `onError` - Playback error occurred

## 🛡️ Security Considerations

### Frame Embedding Policy

The wrapper sets `frame-ancestors *` in the CSP header, which allows **any origin** to embed the wrapper in an iframe. This is necessary for Cordova/Capacitor apps (which use `file://`, `ionic://`, or `capacitor://` protocols) to work.

**Trade-off:** This makes the wrapper embeddable by any website. However, since:
- The wrapper only displays YouTube content (not your sensitive data)
- YouTube already allows universal embedding
- The wrapper requires a valid YouTube video ID

...this is generally an acceptable security posture for this use case.

**If you need stricter controls:** Modify the `frame-ancestors` directive in `netlify.toml` to whitelist specific domains.

### Content Security Policy (CSP)

The wrapper enforces a strict CSP that:
- Only allows YouTube embeds from `youtube.com` and `youtube-nocookie.com`
- Only loads scripts and styles from self and YouTube
- Only loads images from YouTube CDN
- Blocks all other third-party content

## 🔧 Local Development

### Prerequisites

- Node.js 14+ (for dev server)
- A modern browser

### Run Locally

```bash
# Clone the repository
git clone https://github.com/yourusername/yt-embed-wrapper.git
cd yt-embed-wrapper

# Start local dev server
npm run dev

# Open http://localhost:5173
```

### Alternative: Using Python

```bash
cd public
python3 -m http.server 8080
```

### Test the Wrapper

1. Visit `http://localhost:5173`
2. Append a video ID to the URL: `http://localhost:5173/?id=dQw4w9WgXcQ`

## 🚀 Deployment

### Option 1: Netlify (Recommended)

**Via Git:**
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login to Netlify
netlify login

# Deploy
netlify init
netlify deploy --prod
```

**Via Drag & Drop:**
1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag the `public` folder
3. Done!

### Option 2: Cloudflare Pages

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/) → Pages
2. Create a new project from Git repository
3. Configure build settings:
   - **Build command:** (leave empty)
   - **Build output directory:** `public`
4. Deploy!

**Custom Headers for Cloudflare Pages:**

Create a `public/_headers` file or use a `_worker.js` for more control.

### Option 3: Vercel

```bash
npm install -g vercel
vercel --prod
```

**vercel.json:**
```json
{
  "public": true,
  "rewrites": [{ "source": "/(.*)", "destination": "/public/$1" }]
}
```

### Option 4: GitHub Pages

```bash
# Build (not needed for static site)
# Just push the public folder

gh-pages -d public
```

## 📱 Mobile App Integration

### Cordova Example

```html
<!-- index.html -->
<div class="video-container">
  <iframe
    id="yt-player"
    src="https://your-site.netlify.app/?id=dQw4w9WgXcQ&autoplay=1&mute=1"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; fullscreen"
    allowfullscreen
  ></iframe>
</div>

<style>
  .video-container {
    position: relative;
    width: 100%;
    padding-bottom: 56.25%; /* 16:9 aspect ratio */
    height: 0;
  }
  
  .video-container iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border: 0;
  }
</style>
```

### Capacitor Example (React)

```jsx
import React from 'react';

function YouTubePlayer({ videoId }) {
  return (
    <div className="video-wrapper">
      <iframe
        src={`https://your-site.netlify.app/?id=${videoId}&autoplay=1&mute=1`}
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; fullscreen"
        allowFullScreen
        style={{
          width: '100%',
          height: '100%',
          border: 0
        }}
      />
    </div>
  );
}

export default YouTubePlayer;
```

### Capacitor Example (Vue)

```vue
<template>
  <div class="video-wrapper">
    <iframe
      :src="embedUrl"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; fullscreen"
      allowfullscreen
    />
  </div>
</template>

<script>
export default {
  props: {
    videoId: {
      type: String,
      required: true
    }
  },
  computed: {
    embedUrl() {
      return `https://your-site.netlify.app/?id=${this.videoId}&autoplay=1&mute=1`;
    }
  }
}
</script>

<style scoped>
.video-wrapper {
  position: relative;
  padding-bottom: 56.25%;
  height: 0;
  overflow: hidden;
}

.video-wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 0;
}
</style>
```

## 🐛 Troubleshooting

### Video Won't Play on iOS

**Issue:** Video doesn't autoplay on iOS devices.

**Solutions:**
1. ✅ Add `mute=1` parameter (iOS requires muted autoplay)
2. ✅ Ensure iframe has `allow="autoplay"` attribute
3. ✅ Check that `playsinline=1` is in the YouTube URL (automatically added)

### "Protocols Must Match" Error Still Occurs

**Issue:** Still seeing protocol mismatch errors.

**Solutions:**
1. ✅ Verify your wrapper is served over HTTPS (not HTTP)
2. ✅ Check that your app is iframing the wrapper (not YouTube directly)
3. ✅ Clear app cache and rebuild

### CSP frame-ancestors Error on iOS

**Issue:** "Refused to load [...] because it does not appear in the frame-ancestors directive of the Content Security Policy."

**Solutions:**
1. ✅ Ensure `X-Frame-Options` header is NOT set (conflicts with CSP)
2. ✅ Verify CSP has `frame-ancestors *;` (note the semicolon at the end)
3. ✅ Redeploy to Netlify after updating `netlify.toml`
4. ✅ Clear browser/app cache and test again
5. ✅ Check headers in browser DevTools → Network → Response Headers

**Note:** iOS WebKit is particularly strict about CSP. `X-Frame-Options` and CSP `frame-ancestors` cannot coexist - always use only CSP's `frame-ancestors` for modern browsers.

### Error 153 (YouTube API Error)

**Issue:** YouTube shows "Error loading player: Error Code 153"

**Solutions:**
1. ✅ Check that `origin` parameter matches your wrapper's domain
2. ✅ Verify referrer policy is set correctly
3. ✅ Ensure CSP headers allow YouTube iframe

### Video Blocked by Ad Blocker

**Issue:** Video doesn't load due to ad blocker.

**Solutions:**
1. ✅ Whitelist your wrapper domain in ad blocker
2. ✅ Use `youtube-nocookie.com` domain (privacy-focused)
3. ✅ Inform users to disable ad blockers

### PostMessage Not Working

**Issue:** Events not received from wrapper.

**Solutions:**
1. ✅ Check cross-origin restrictions (same-origin policy)
2. ✅ Verify event listener is registered before iframe loads
3. ✅ Use `'*'` as target origin in postMessage for development

### Corporate Network Blocking YouTube

**Issue:** Video doesn't load on corporate networks.

**Solutions:**
1. ✅ Check network firewall rules
2. ✅ Try `youtube-nocookie.com` domain
3. ✅ Contact IT department to whitelist YouTube

## 📊 Browser Support

| Browser | Support | Notes |
|---------|---------|-------|
| Safari iOS 12+ | ✅ Full | Primary use case |
| Chrome Android | ✅ Full | Works perfectly |
| Safari macOS | ✅ Full | Desktop support |
| Chrome Desktop | ✅ Full | Desktop support |
| Firefox | ✅ Full | Desktop support |
| Edge | ✅ Full | Desktop support |
| WKWebView | ✅ Full | Cordova/Capacitor |
| UIWebView | ⚠️ Limited | Deprecated, use WKWebView |

## 🔒 Privacy & Analytics

This wrapper **does not include any analytics or tracking** by default. All traffic goes directly to YouTube, subject to YouTube's privacy policy.

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built for the mobile developer community
- Inspired by common iOS WebView embedding issues
- Thanks to all contributors and users!

## 📧 Support

- 🐛 **Issues:** [GitHub Issues](https://github.com/yourusername/yt-embed-wrapper/issues)
- 💬 **Discussions:** [GitHub Discussions](https://github.com/yourusername/yt-embed-wrapper/discussions)
- 📧 **Email:** your.email@example.com

## 🔗 Related Projects

- [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference)
- [Capacitor](https://capacitorjs.com/)
- [Cordova](https://cordova.apache.org/)

---

Made with ❤️ for developers struggling with iOS WebView embedding
