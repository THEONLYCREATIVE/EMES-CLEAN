# 📦 GS1 Inventory Scanner PWA

A Progressive Web App for scanning GS1 barcodes with permanent IndexedDB storage, automatic quantity merging, and custom CSV export format.

## ✨ Key Features

### 🎯 Core Functionality
- **Permanent Storage**: Uses IndexedDB for persistent data storage across sessions
- **GS1 Barcode Scanning**: Camera-based barcode scanning with auto-parsing
- **Auto-Fill**: Automatically extracts Name, Expiry Date, and Batch from GS1 barcodes
- **Smart Merge**: Automatically merges quantities for duplicate items (same barcode + batch + expiry)
- **Custom Export**: Exports data in your exact format: RMS, DESCRIPTION, BARCODE, QNTY, EXPIRY, BATCH, LOCATION NO, LOCATION NAME

### 📱 PWA Features
- **Offline Support**: Works without internet connection
- **Installable**: Add to home screen on mobile devices
- **Responsive**: Works on all screen sizes

## 🚀 Quick Start

### Option 1: Local Hosting
1. Place all files in a folder:
   - `inventory-scanner.html`
   - `manifest.json`
   - `sw.js`

2. Serve via local web server:
   ```bash
   # Python 3
   python -m http.server 8000
   
   # Node.js (if you have http-server installed)
   npx http-server
   ```

3. Open browser: `http://localhost:8000/inventory-scanner.html`

### Option 2: Deploy to Web
Upload all files to any web hosting service (GitHub Pages, Netlify, Vercel, etc.)

## 📖 Usage Guide

### Scanning Workflow

1. **Scan Barcode**
   - Click "📸 Scan Barcode"
   - Grant camera permission
   - Point camera at GS1 barcode
   - System auto-fills: Name, Expiry, Batch

2. **Auto-Fill Behavior**
   - **Name**: Extracted from GTIN (AI 01)
   - **Expiry**: Parsed from date field (AI 17) - format YYMMDD
   - **Batch**: Extracted from batch/lot field (AI 10)
   - **Quantity**: Defaults to 1

3. **Manual Entry** (Optional)
   - Click "✏️ Manual Entry"
   - Fill in any missing fields:
     - RMS (Product Code)
     - Location No
     - Location Name
   - Adjust quantity if needed

4. **Add Item**
   - Click "➕ Add Item"
   - System checks for duplicates
   - If found: Quantities merge automatically
   - If new: Item added to database

### Auto-Merge Logic

Items are considered duplicates when ALL match:
- Same Barcode
- Same Batch Number
- Same Expiry Date

**Example:**
```
Scan 1: Barcode 123, Batch A, Expiry 2025-12-31, Qty 5
Scan 2: Barcode 123, Batch A, Expiry 2025-12-31, Qty 3
Result: Single entry with Qty 8 ✅

Scan 3: Barcode 123, Batch B, Expiry 2025-12-31, Qty 2
Result: Separate entry (different batch) ✅
```

### Export Format

CSV export maintains your exact format:

```csv
RMS,DESCRIPTION,BARCODE,QNTY,EXPIRY,BATCH,LOCATION NO,LOCATION NAME
30762,"Product Name",01234567890123,10,2025-12-31,LOT123,30762,BOOTS OASIS
```

**Field Mapping:**
- RMS → Custom product code
- DESCRIPTION → Product name
- BARCODE → Scanned barcode
- QNTY → Total quantity (after merges)
- EXPIRY → Expiry date (YYYY-MM-DD)
- BATCH → Batch/lot number
- LOCATION NO → Location code
- LOCATION NAME → Location description

## 🔍 GS1 Barcode Support

### Supported Application Identifiers (AI)

| AI | Description | Length | Example |
|----|-------------|--------|---------|
| 01 | GTIN (Product ID) | 14 digits | 01234567890123 |
| 10 | Batch/Lot | Variable | LOT12345 |
| 17 | Expiry Date | 6 digits (YYMMDD) | 251231 |
| 21 | Serial Number | Variable | SN98765 |

### Example GS1 Barcode

```
0112345678901234102023051721SN67890
```

Parsed as:
- **01** 12345678901234 → GTIN
- **10** 20230517 → Batch
- **21** SN67890 → Serial

## 💾 Data Persistence

### IndexedDB Structure

**Database**: `InventoryDB`  
**Store**: `items`  
**Schema**:
```javascript
{
  id: 1,                           // Auto-increment
  rms: "30762",                    // Product code
  description: "Product Name",     // Name
  barcode: "01234567890123",       // Scanned code
  quantity: 10,                    // Total qty
  expiry: "2025-12-31",           // Date
  batch: "LOT123",                // Batch number
  locationNo: "30762",            // Location code
  locationName: "BOOTS OASIS",    // Location name
  timestamp: "2025-02-17T..."     // When added
}
```

### Data Persistence
- ✅ Survives browser refresh
- ✅ Survives browser close/reopen
- ✅ Survives device restart
- ⚠️ Cleared if browser cache/data is cleared
- ⚠️ Domain-specific (won't transfer between domains)

## 🎨 UI Features

### Dashboard Stats
- **Total Items**: Unique product entries
- **Total Quantity**: Sum of all quantities

### Table View
- Sortable columns
- Delete individual items
- Clear all items
- Real-time updates

### Camera Scanner
- Live video preview
- Animated scan line
- Auto-stop on successful scan
- Mobile-optimized (rear camera default)

## 📱 Mobile Installation

### iOS (Safari)
1. Open the app in Safari
2. Tap Share button
3. Select "Add to Home Screen"
4. Tap "Add"

### Android (Chrome)
1. Open the app in Chrome
2. Tap menu (⋮)
3. Select "Add to Home Screen"
4. Tap "Add"

## 🔧 Customization

### Modify Export Format
Edit the export function in the HTML file:

```javascript
const headers = ['RMS', 'DESCRIPTION', 'BARCODE', 'QNTY', ...];
const rows = items.map(item => [
  item.rms || '',
  item.description,
  // Add/remove fields as needed
]);
```

### Change Default Values
```javascript
document.getElementById('locationNo').value = 'YOUR_DEFAULT';
document.getElementById('locationName').value = 'YOUR_STORE';
```

### Styling
All styles are in the `<style>` section - modify colors, fonts, layouts as needed.

## 🐛 Troubleshooting

### Camera Not Working
- **Check permissions**: Allow camera access in browser settings
- **HTTPS required**: Camera API requires secure context (https:// or localhost)
- **Mobile**: Ensure camera permission granted to browser

### Barcode Not Scanning
- **Lighting**: Ensure good lighting on barcode
- **Distance**: Keep camera 6-12 inches from barcode
- **GS1 format**: Non-GS1 barcodes will scan but won't auto-parse
- **Manual entry**: Use "Manual Entry" as fallback

### Data Not Persisting
- **Check browser**: IndexedDB supported in all modern browsers
- **Private mode**: Data won't persist in incognito/private mode
- **Storage quota**: Clear old data if storage full

### Export Not Working
- **Check data**: Ensure items exist in database
- **Pop-up blocker**: Allow downloads from the site
- **Mobile**: Download will appear in browser downloads

## 🔒 Privacy & Security

- **Local Storage**: All data stored locally in browser
- **No Cloud**: No data sent to external servers
- **Offline**: Works completely offline
- **No Tracking**: No analytics or tracking code

## 📝 Technical Details

### Technologies Used
- **HTML5** - Structure
- **CSS3** - Styling (no frameworks)
- **Vanilla JavaScript** - Logic
- **IndexedDB API** - Persistent storage
- **MediaDevices API** - Camera access
- **jsQR Library** - Barcode decoding
- **Service Worker** - Offline support

### Browser Compatibility
- ✅ Chrome/Edge (Desktop & Mobile)
- ✅ Safari (iOS & macOS)
- ✅ Firefox (Desktop & Mobile)
- ⚠️ IE11 (Not supported - requires modern browser)

### File Size
- HTML: ~18KB
- Manifest: ~1KB
- Service Worker: ~1KB
- **Total**: ~20KB (very lightweight!)

## 🎯 Use Cases

1. **Retail Inventory**: Track stock in stores
2. **Warehouse Management**: Receive and track shipments
3. **Pharmacy**: Manage medications with batch/expiry
4. **Food Industry**: Track products with expiration dates
5. **Distribution**: Log incoming/outgoing items

## 📊 Future Enhancements

Potential features to add:
- [ ] Export to Excel (.xlsx)
- [ ] Import from CSV
- [ ] Barcode label printing
- [ ] Multi-location support
- [ ] User authentication
- [ ] Cloud sync option
- [ ] Batch edit mode
- [ ] Search and filter
- [ ] Reports and analytics

## 🤝 Support

For issues or questions:
1. Check troubleshooting section
2. Verify GS1 barcode format
3. Test with manual entry first
4. Check browser console for errors

## 📄 License

Free to use and modify for your needs.

---

**Version**: 1.0  
**Last Updated**: February 2025  
**Built with**: ❤️ for efficient inventory management
