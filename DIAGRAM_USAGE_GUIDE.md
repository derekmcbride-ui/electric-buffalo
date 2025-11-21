# Architecture Diagram Usage Guide

You now have a professional, vector-based architecture diagram with ParentSquare branding!

---

## 📁 Files Created

- **`architecture_diagram.svg`** - Scalable vector graphic (can be resized without quality loss)
- **`architecture_diagram.png`** - High-resolution PNG (2400px width, ready to insert anywhere)
- **`architecture_diagram_viewer.html`** - Quick viewer with instructions

---

## 🎯 How to Use the Diagram

### Option 1: Insert SVG into Word/PowerPoint (Recommended)

**Word:**
1. Open your Word document
2. Click where you want the diagram
3. **Insert → Pictures → Picture from File**
4. Choose `architecture_diagram.svg`
5. Resize as needed (won't lose quality!)

**PowerPoint:**
1. Open your presentation
2. Click the slide where you want the diagram
3. **Insert → Pictures → Picture from File**
4. Choose `architecture_diagram.svg`
5. Resize to fit your slide

✅ **Benefits:** Scales perfectly, editable colors, small file size

---

### Option 2: Convert to PNG (For Compatibility)

If you need a PNG file (some older systems don't support SVG):

**Method A: Using Preview (Mac)**
1. Right-click `architecture_diagram.svg`
2. **Open With → Preview**
3. **File → Export**
4. Format: **PNG**
5. Resolution: **300 DPI** (for print quality)
6. Save as `architecture_diagram.png`

**Method B: Using Browser**
1. Open `architecture_diagram_viewer.html` in browser
2. Right-click the diagram
3. **Save Image As...**
4. Save as PNG

**Method C: Using Command Line (Mac/Linux)**
```bash
cd "/Users/derek/Claude/IM Dashboard"
# If you have ImageMagick installed:
convert -density 300 architecture_diagram.svg architecture_diagram.png

# Or using rsvg-convert:
rsvg-convert -w 2400 architecture_diagram.svg -o architecture_diagram.png
```

---

### Option 3: Screenshot (Quick & Easy)

1. Open `architecture_diagram_viewer.html` in browser
2. Press **Cmd+Shift+4** (Mac) or **Windows+Shift+S** (Windows)
3. Select the diagram area
4. Paste into your document

✅ **Benefits:** Instant, no conversion needed

---

### Option 4: Print to PDF

1. Open `architecture_diagram_viewer.html` in browser
2. Press **Cmd+P** (Mac) or **Ctrl+P** (Windows)
3. Destination: **Save as PDF**
4. Save as `architecture_diagram.pdf`
5. Now you have a PDF version you can insert anywhere

---

## 🎨 Diagram Features

✅ **ParentSquare Branding:**
- Green (#85ad33) for central dashboard
- Blue (#7bbaf0) for integrations
- Light green (#e5f1d4) for HubSpot (primary)
- Clean, professional design

✅ **Shows:**
- Central IM Dashboard as hub
- HubSpot as primary bidirectional integration
- Read-only integrations (Jira, Zendesk, ParentSquare Usage)
- Future Phase 3 integrations (dashed)
- Clear data flow arrows
- Legend explaining colors

✅ **Perfect For:**
- Executive presentations
- Technical documentation
- Team alignment meetings
- Stakeholder demos

---

## 💡 Pro Tips

### Editing the SVG (If Needed)

If you need to modify colors or text:

1. **Open in text editor** (SVG is just XML)
2. Find the color you want to change (search for hex codes like `#85ad33`)
3. Replace with new hex code
4. Save and reload in browser

**Or use online tools:**
- **Figma** (free) - Import SVG, edit visually
- **Inkscape** (free) - Full-featured SVG editor
- **Vectr** (free, online) - Simple SVG editing

### Inserting in Google Docs

Google Docs doesn't support SVG directly:
1. Convert to PNG first (see Option 2 above)
2. Or screenshot and paste
3. Or convert to PDF and link it

### Printing

The diagram is 800x600px by default. For printing:
- Use the PDF export method for best quality
- Or convert to PNG at 300 DPI minimum

---

## 🚀 Quick Start for Monday Presentation

**For PowerPoint:**
1. Open your presentation
2. Create a new slide with title "Architecture Overview"
3. Insert → Pictures → `architecture_diagram.png` (or use .svg for scalability)
4. Resize to fill most of the slide
5. Done! ✅

**For Word:**
1. Open `IM_Dashboard_Executive_Summary_Branded.docx`
2. Navigate to "Architecture Overview" section
3. Delete the ASCII art text
4. Insert → Pictures → `architecture_diagram.png` (or use .svg for scalability)
5. Center the image
6. Done! ✅

💡 **Tip:** Use the PNG for maximum compatibility, or use the SVG if you need to resize it to different sizes without quality loss.

**For Web/HTML:**
Already embedded in:
- `IM_Dashboard_Presentation.html`
- `architecture_diagram_viewer.html`

---

## 📊 Technical Details

- **Format:** SVG (Scalable Vector Graphics)
- **Size:** 800x640px (but scales infinitely!)
- **Colors:** ParentSquare brand palette
- **Font:** Arial (universally supported)
- **File size:** ~5KB (very small!)

---

## ❓ Troubleshooting

**"The image won't insert in Word"**
- Make sure you're using Insert → Pictures (not Insert → Online Pictures)
- Try converting to PNG first (see Option 2)

**"The colors look different when inserted"**
- This is normal for different programs
- Colors are correct in the SVG file itself
- For exact color matching, use the PNG conversion at 300 DPI

**"I need to change something in the diagram"**
- Let me know what you want changed and I can update the SVG!
- Or edit the SVG file directly in a text editor (it's just XML)

---

**Created for IM Dashboard Project**
**Derek McBride • ParentSquare**
