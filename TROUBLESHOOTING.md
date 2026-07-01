# Troubleshooting Guide

## How to Run the Game

### Option 1: Direct Browser Open (Simplest)
1. Double-click `test.html` in Windows Explorer
2. This will open in your default browser with error logging
3. Check the page for any error messages

### Option 2: Using a Local Server (Recommended)
If you have Python installed:
```bash
# Python 3
python -m http.server 8080

# Then open: http://localhost:8080
```

If you have Node.js/npm installed:
```bash
npm install
npm start
```

### Option 3: Direct File Open
1. Right-click `index.html`
2. Choose "Open with" → Your browser (Chrome, Firefox, Edge)

## Common Issues

### Issue 1: "File not loading" or blank screen
**Cause**: Browser security restrictions on local files
**Solution**: 
- Use `test.html` instead - it has error logging
- Or use a local web server (see Option 2 above)
- Check browser console (F12) for errors

### Issue 2: "Phaser is not defined"
**Cause**: CDN not loading or no internet connection
**Solution**: 
- Check your internet connection
- Try refreshing the page
- Check browser console for network errors

### Issue 3: Game freezes on restart
**Cause**: Fixed in latest version
**Solution**: Make sure you're using the latest GameScene.js

### Issue 4: Bullets don't move
**Cause**: Fixed in latest version
**Solution**: Clear browser cache (Ctrl+Shift+Delete) and reload

## Checking for Errors

1. Open the game in your browser
2. Press F12 to open Developer Tools
3. Click the "Console" tab
4. Look for red error messages
5. Share any errors you see

## Browser Compatibility

Tested and working on:
- Chrome 90+
- Firefox 88+
- Edge 90+
- Safari 14+

## File Structure Check

Your project should have:
```
windsurf-project/
├── index.html
├── test.html (for debugging)
├── run.bat (Windows shortcut)
├── src/
│   ├── GameScene.js
│   └── main.js
├── package.json
└── README.md
```

If any files are missing, that could be the issue.
