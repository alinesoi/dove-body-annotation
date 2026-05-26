# Confident Me - Body Annotation Activity: Development Guide

## Project Overview

An interactive educational activity for Dove's "Confident Me" program where students identify and annotate appearance traits on a body figure. Built as a single-page HTML/CSS/JS app deployed on GitHub Pages.

**Live URL**: https://alinesoi.github.io/dove-body-annotation/
**Repo**: https://github.com/alinesoi/dove-body-annotation

---

## Architecture: Hybrid Image + HTML Approach

### Background Image (Static/Decorative Only)
The mockup image (`mockup_bg_noslots.png`) provides purely decorative elements:
- Mint/teal background gradient
- Body figure illustration (child-like, teal fill, navy outline, hotspot dots)
- Title "Confident Me" + "Appearance Ideals" banner
- "Safe Space" badge (top-right)
- Home button visual (top-left)
- Decorative stars, hearts, and dots (right side and scattered)
- White container rectangle
- Golden-bordered input bar (border only, no text)

### HTML Elements (All Interactive/Dynamic)
Everything the user interacts with or that changes dynamically is pure HTML:
- **Sidebar cards**: "Your Mission" + "Progress" (teal headers, white bodies)
- **8 annotation slots**: Dashed-border rounded rectangles (4 left, 4 right)
- **Input text**: Placeholder "What traits come to mind?" (2vw font, centered)
- **Progress counter**: Number + "traits found" + hint text
- **Body hotspot zones**: 9 invisible click targets (head, shoulders, chest, arms x2, abs, hips, legs, feet)
- **Annotation popup**: Appears on body click with input + "Add" button
- **Send/Done buttons**: Overlaid on the golden bar area

### Why This Approach
We started with full CSS recreation (v1-v23) but couldn't match the AI-generated mockup precisely. We then tried using the mockup as a background image with transparent overlays (v24-v28), but alignment issues persisted. The final approach removes all interactive elements from the image using GPT image editing, then renders them in HTML. This eliminates alignment problems since HTML elements ARE their own borders.

---

## Image Editing Pipeline

We used Azure OpenAI `gpt-image-2-1` to progressively clean the mockup:

1. **Original mockup** (`ui_concept_6.png`): AI-generated concept with all elements
2. **Remove mascot** (`mockup_bg_no_mascot.png`): Blue monster character removed from bottom-left
3. **Remove input text** (`mockup_bg_clean.png`): "What traits come to mind?" placeholder removed
4. **Remove progress text** (`mockup_bg_final.png`): "0", "traits found", hint text removed from Progress card
5. **Remove sidebar cards** (`mockup_bg_nosidebar.png`): Both "Your Mission" and "Progress" cards removed
6. **Remove slots** (`mockup_bg_noslots.png`): All 8 dashed annotation slot borders removed

Each step used the Azure OpenAI image edit API:
```python
resp = httpx.post(
    f"{endpoint}/openai/deployments/gpt-image-2-1/images/edits?api-version=...",
    files={"image": ("img.png", img_bytes, "image/png")},
    data={"prompt": "Remove X, fill with matching background...", "size": "1536x1024"}
)
```

---

## Element Positioning

Positions were detected using Azure OpenAI GPT-4o vision analysis:

### Annotation Slots (GPT-4o detected)
| Slot | Position | Region |
|------|----------|--------|
| slot-0 | left:21.4%, top:23.1% | Head (left) |
| slot-1 | left:65.2%, top:23.1% | Head (right) |
| slot-2 | left:21.4%, top:36.4% | Shoulders/Chest (left) |
| slot-3 | left:65.2%, top:36.4% | Shoulders/Chest (right) |
| slot-4 | left:21.4%, top:51% | Abs/Hips (left) |
| slot-5 | left:65.2%, top:51% | Abs/Hips (right) |
| slot-6 | left:21.4%, top:66.8% | Legs/Feet (left) |
| slot-7 | left:65.2%, top:66.8% | Legs/Feet (right) |

All slots: width:14.2%, height:7%, border-radius:14px

### Body Figure (GPT-4o detected)
- Bounding box: left=39.6%, top=14.5%, width=24.2%, height=66.4%
- 9 hotspot zones covering head, shoulders, chest, arms (x2), abs, hips, legs, feet

### Input Bar
- Text overlay: left:26%, top:86%, width:48%, height:5%
- Font: 2vw, centered, Nunito

### Sidebar Cards
- Container: left:1.5%, top:16%, width:13%, height:50%
- "Your Mission" card: teal header, white body with description
- "Progress" card: teal header, large number, "traits found" label, hint text

---

## Interactive Features

### 1. Trait Input (Bottom Bar)
- User types a trait in the golden input bar
- Sent to Azure OpenAI GPT-5.4 for classification
- AI returns: `{"trait": "Beautiful Eyes", "region": "head"}`
- Trait placed in the corresponding slot with animation

### 2. Body Part Clicking (Hotspots)
- 9 invisible zones overlaid on the body figure
- Click opens a popup: "Add trait for [Region]"
- User types trait, clicks Add
- Trait placed in the slot for that region

### 3. Smile Zoom
- Typing "smile" triggers a special zoom overlay
- Shows 3 smile variation images to choose from
- Or user can type a custom description

### 4. Progress Tracking
- Counter updates with each trait added
- Milestone toasts at 3, 5, 10, 15, 20 traits
- Confetti bursts on milestones

### 5. Sound Effects
- Web Audio API synthesized sounds
- Pop, ding, select, click, celebration, whoosh, error

### 6. Done Screen
- Shows all collected traits with count
- Confetti celebration
- "Back to Activity" button

---

## Canvas Scaling

The game uses a fixed 3:2 aspect ratio canvas:
```css
.canvas {
  width: min(100vw, 150vh);
  height: min(100vh, 66.67vw);
  background: url('images/mockup_bg_noslots.png') center/100% 100% no-repeat;
}
```
This ensures the canvas always fills the viewport while maintaining the image's aspect ratio. All overlay positions use percentages relative to this canvas.

---

## API Configuration

- **Endpoint**: Azure OpenAI (`nesoi-resource.openai.azure.com`)
- **Model**: GPT-5.4 for trait classification
- **Image editing**: GPT-image-2-1 for mockup modifications

---

## Version History

| Version | Key Change |
|---------|-----------|
| v1-v20 | Full CSS recreation attempts |
| v21-v23 | Bigger decorations, dove logo, white rectangle |
| v24 | Switched to mockup-as-background approach |
| v25 | Removed mascot via GPT image editing |
| v26-v28 | GPT-4o vision-based overlay positioning |
| v29 | Removed green hover rectangles on hotspots |
| v31 | Removed baked-in input text from image |
| v33-v35 | Input text size and position refinements |
| v39 | Removed baked-in progress text from image |
| v44 | Removed sidebar cards from image, HTML-only |
| v45 | Removed slot borders from image, HTML-only (Strategy 2) |
| v46-v51 | Popup styling refinements |
