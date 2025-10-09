<script setup>
import { ref, computed, onMounted } from 'vue';
import QrcodeVue from 'qrcode.vue';
import { PDFDocument, rgb, StandardFonts } from 'pdf-lib';

// --- Constants for Page Sizes in pt ---
const PAGE_SIZES = {
  A4: { width: 595, height: 842 },
  Letter: { width: 612, height: 792 },
};

// --- Reactive State ---
const qrValue = ref('https://github.com/copilot');
const qrSize = ref(150);
const qrPosition = ref({ x: 50, y: 50 });

const additionalTexts = ref([
  { 
    id: 1, 
    text: 'Sample Text', 
    position: { x: 50, y: 220 },
    style: {
      fontSize: 16,
      fontFamily: 'Helvetica',
      color: '#000000',
      bold: false,
      italic: false,
      underline: false,
      textAlign: 'left'
    }
  }
]);

const pageSize = ref('A4');
const pageOrientation = ref('portrait');
const customSize = ref({ width: 595, height: 842 });

// --- Drag and Drop State ---
const dragState = ref({
  isDragging: false,
  target: null, // 'qr' or text id
  offsetX: 0,
  offsetY: 0,
});

// --- Computed Properties ---
const previewSheetStyle = computed(() => {
  let w, h;
  if (pageSize.value === 'Custom') {
    w = customSize.value.width;
    h = customSize.value.height;
  } else {
    w = PAGE_SIZES[pageSize.value].width;
    h = PAGE_SIZES[pageSize.value].height;
  }

  if (pageOrientation.value === 'landscape') {
    [w, h] = [h, w]; // Swap width and height
  }

  // Scale to fit preview area while maintaining aspect ratio
  const containerWidth = 600; // Example fixed width for preview area
  const scale = containerWidth / w;
  
  return {
    width: `${w * scale}px`,
    height: `${h * scale}px`,
    position: 'relative',
    backgroundColor: 'white',
    boxShadow: '0 0 10px rgba(0,0,0,0.1)',
    margin: 'auto',
  };
});

const scaleRatio = computed(() => {
    let w = pageSize.value === 'Custom' ? customSize.value.width : PAGE_SIZES[pageSize.value].width;
    if (pageOrientation.value === 'landscape' && pageSize.value !== 'Custom') {
        w = PAGE_SIZES[pageSize.value].height;
    }
    const containerWidth = 600;
    return containerWidth / w;
});


// --- Functions ---
function getFont(pdfDoc, fontFamily, bold, italic) {
  const fontMap = {
    'Helvetica': {
      normal: StandardFonts.Helvetica,
      bold: StandardFonts.HelveticaBold,
      italic: StandardFonts.HelveticaOblique,
      boldItalic: StandardFonts.HelveticaBoldOblique
    },
    'Times': {
      normal: StandardFonts.TimesRoman,
      bold: StandardFonts.TimesRomanBold,
      italic: StandardFonts.TimesRomanItalic,
      boldItalic: StandardFonts.TimesRomanBoldItalic
    },
    'Courier': {
      normal: StandardFonts.Courier,
      bold: StandardFonts.CourierBold,
      italic: StandardFonts.CourierOblique,
      boldItalic: StandardFonts.CourierBoldOblique
    }
  };

  const fonts = fontMap[fontFamily] || fontMap['Helvetica'];
  let fontType;
  
  if (bold && italic) {
    fontType = fonts.boldItalic;
  } else if (bold) {
    fontType = fonts.bold;
  } else if (italic) {
    fontType = fonts.italic;
  } else {
    fontType = fonts.normal;
  }
  
  return pdfDoc.embedFont(fontType);
}

function hexToRgb(hex) {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result ? {
    r: parseInt(result[1], 16) / 255,
    g: parseInt(result[2], 16) / 255,
    b: parseInt(result[3], 16) / 255
  } : { r: 0, g: 0, b: 0 };
}

function wrapText(text, font, fontSize, maxWidth) {
  const words = text.split(' ');
  const lines = [];
  let currentLine = '';

  for (const word of words) {
    const testLine = currentLine ? `${currentLine} ${word}` : word;
    const testWidth = font.widthOfTextAtSize(testLine, fontSize);
    
    if (testWidth <= maxWidth) {
      currentLine = testLine;
    } else {
      if (currentLine) {
        lines.push(currentLine);
        currentLine = word;
      } else {
        // Se uma única palavra é maior que a largura máxima, precisamos quebrá-la
        const chars = word.split('');
        let charLine = '';
        for (const char of chars) {
          const testCharLine = charLine + char;
          if (font.widthOfTextAtSize(testCharLine, fontSize) <= maxWidth) {
            charLine = testCharLine;
          } else {
            if (charLine) lines.push(charLine);
            charLine = char;
          }
        }
        if (charLine) currentLine = charLine;
      }
    }
  }
  
  if (currentLine) {
    lines.push(currentLine);
  }
  
  return lines;
}

function addText() {
  const newId = additionalTexts.value.length > 0 ? Math.max(...additionalTexts.value.map(t => t.id)) + 1 : 1;
  additionalTexts.value.push({ 
    id: newId, 
    text: 'New Text', 
    position: { x: 50, y: 250 },
    style: {
      fontSize: 16,
      fontFamily: 'Helvetica',
      color: '#000000',
      bold: false,
      italic: false,
      underline: false,
      textAlign: 'left'
    }
  });
}

function removeText(id) {
  additionalTexts.value = additionalTexts.value.filter(t => t.id !== id);
}

async function generatePdf() {
  try {
    // 1. Get Page Dimensions
    let pageWidth, pageHeight;
    if (pageSize.value === 'Custom') {
      pageWidth = customSize.value.width;
      pageHeight = customSize.value.height;
    } else {
      pageWidth = PAGE_SIZES[pageSize.value].width;
      pageHeight = PAGE_SIZES[pageSize.value].height;
    }

    if (pageOrientation.value === 'landscape') {
      [pageWidth, pageHeight] = [pageHeight, pageWidth];
    }

    // 2. Create PDF
    const pdfDoc = await PDFDocument.create();
    const page = pdfDoc.addPage([pageWidth, pageHeight]);
    const scale = scaleRatio.value;

    // 3. Handle QR Code
    const qrCanvas = document.querySelector('.draggable-qr canvas');
    if (qrCanvas) {
      const qrPngDataUrl = qrCanvas.toDataURL('image/png');
      const qrPngImageBytes = await fetch(qrPngDataUrl).then(res => res.arrayBuffer());
      const qrImage = await pdfDoc.embedPng(qrPngImageBytes);
      
      const pdfQrSize = qrSize.value / scale;
      const pdfQrX = qrPosition.value.x / scale;
      const pdfQrY = pageHeight - (qrPosition.value.y / scale) - pdfQrSize;

      page.drawImage(qrImage, {
        x: pdfQrX,
        y: pdfQrY,
        width: pdfQrSize,
        height: pdfQrSize,
      });
    }

    // 4. Handle Texts
    for (const textItem of additionalTexts.value) {
      const font = await getFont(pdfDoc, textItem.style.fontFamily, textItem.style.bold, textItem.style.italic);
      const fontSize = textItem.style.fontSize / scale;
      const pdfTextX = textItem.position.x / scale;
      const pdfTextY = pageHeight - (textItem.position.y / scale) - fontSize;
      
      const { r, g, b } = hexToRgb(textItem.style.color);

      // Calculate maximum width available for text (from current position to page edge)
      const maxWidth = pageWidth - pdfTextX - 20; // 20pt margin from right edge
      
      // Wrap text into multiple lines if necessary
      const lines = wrapText(textItem.text, font, fontSize, maxWidth);
      
      // Draw each line with proper alignment
      lines.forEach((line, index) => {
        const lineY = pdfTextY - (index * fontSize * 1.2); // 1.2 is line height multiplier
        const lineWidth = font.widthOfTextAtSize(line, fontSize);
        
        let alignedX = pdfTextX;
        
        // Calculate X position based on text alignment
        if (textItem.style.textAlign === 'center') {
          alignedX = pdfTextX - (lineWidth / 2);
        } else if (textItem.style.textAlign === 'right') {
          alignedX = pdfTextX - lineWidth;
        }
        // 'left' uses the original pdfTextX position
        
        page.drawText(line, {
          x: alignedX,
          y: lineY,
          size: fontSize,
          font,
          color: rgb(r, g, b),
        });

        // Handle underline for each line with proper thickness and alignment
        if (textItem.style.underline) {
          const underlineThickness = Math.max(1, fontSize / 16); // Scale thickness with font size
          const underlineY = lineY - (fontSize * 0.1); // Position relative to font size
          page.drawLine({
            start: { x: alignedX, y: underlineY },
            end: { x: alignedX + lineWidth, y: underlineY },
            thickness: underlineThickness,
            color: rgb(r, g, b),
          });
        }
      });
    }

    // 5. Save PDF and Create Download Link
    const pdfBytes = await pdfDoc.save();
    const blob = new Blob([pdfBytes], { type: 'application/pdf' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'generated-qr-code.pdf';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(link.href);

  } catch (error) {
    console.error('Failed to generate PDF:', error);
    alert('An error occurred while generating the PDF. See console for details.');
  }
}

// --- Drag and Drop Logic ---
function startDrag(event, targetId) {
  dragState.value.isDragging = true;
  dragState.value.target = targetId;

  const targetElement = event.currentTarget;
  const rect = targetElement.getBoundingClientRect();
  const parentRect = targetElement.parentElement.getBoundingClientRect();

  dragState.value.offsetX = event.clientX - rect.left;
  dragState.value.offsetY = event.clientY - rect.top;

  window.addEventListener('mousemove', onDrag);
  window.addEventListener('mouseup', stopDrag);
}

function onDrag(event) {
  if (!dragState.value.isDragging) return;

  const parentRect = document.querySelector('.preview-sheet').getBoundingClientRect();
  let newX = event.clientX - parentRect.left - dragState.value.offsetX;
  let newY = event.clientY - parentRect.top - dragState.value.offsetY;

  let targetWidth = 0;
  let targetHeight = 0;

  if (dragState.value.target === 'qr') {
    targetWidth = qrSize.value;
    targetHeight = qrSize.value;
  } else {
    const textElement = document.querySelector(`[data-text-id='${dragState.value.target}']`);
    if (textElement) {
      targetWidth = textElement.offsetWidth;
      targetHeight = textElement.offsetHeight;
    }
  }

  // Snap to grid or boundaries
  newX = Math.max(0, Math.min(newX, parentRect.width - targetWidth));
  newY = Math.max(0, Math.min(newY, parentRect.height - targetHeight));

  if (dragState.value.target === 'qr') {
    qrPosition.value = { x: newX, y: newY };
  } else {
    const text = additionalTexts.value.find(t => t.id === dragState.value.target);
    if (text) {
      text.position = { x: newX, y: newY };
    }
  }
}

function stopDrag() {
  dragState.value.isDragging = false;
  dragState.value.target = null;
  window.removeEventListener('mousemove', onDrag);
  window.removeEventListener('mouseup', stopDrag);
}

onMounted(() => {
  // Clean up listeners if component is unmounted
  return () => {
    window.removeEventListener('mousemove', onDrag);
    window.removeEventListener('mouseup', stopDrag);
  };
});

</script>

<template>
  <div id="app-container">
    <aside class="sidebar">
      <h1>QR PDF Generator</h1>

      <div class="card">
        <h2>QR Code</h2>
        <label>QR Content (URL)</label>
        <input type="text" v-model="qrValue" placeholder="https://example.com" />
        <label>QR Size (px on preview)</label>
        <input type="number" v-model.number="qrSize" />
      </div>

      <div class="card">
        <h2>Additional Texts</h2>
        <div v-for="item in additionalTexts" :key="item.id" class="text-config">
          <div class="text-input-row">
            <input type="text" v-model="item.text" />
            <button @click="removeText(item.id)" class="remove-btn">×</button>
          </div>
          
          <div class="text-formatting">
            <div class="format-row">
              <label>Font Size</label>
              <input type="number" v-model.number="item.style.fontSize" min="8" max="72" />
            </div>
            
            <div class="format-row">
              <label>Font Family</label>
              <select v-model="item.style.fontFamily">
                <option value="Helvetica">Helvetica</option>
                <option value="Times">Times New Roman</option>
                <option value="Courier">Courier</option>
              </select>
            </div>
            
            <div class="format-row">
              <label>Color</label>
              <input type="color" v-model="item.style.color" />
            </div>
            
            <div class="format-row">
              <label>Text Align</label>
              <select v-model="item.style.textAlign">
                <option value="left">Left</option>
                <option value="center">Center</option>
                <option value="right">Right</option>
              </select>
            </div>
            
            <div class="format-buttons">
              <button 
                :class="['format-btn', { active: item.style.bold }]"
                @click="item.style.bold = !item.style.bold"
                title="Bold"
              >
                <strong>B</strong>
              </button>
              <button 
                :class="['format-btn', { active: item.style.italic }]"
                @click="item.style.italic = !item.style.italic"
                title="Italic"
              >
                <em>I</em>
              </button>
              <button 
                :class="['format-btn', { active: item.style.underline }]"
                @click="item.style.underline = !item.style.underline"
                title="Underline"
              >
                <u>U</u>
              </button>
            </div>
          </div>
        </div>
        <button @click="addText" class="add-btn">+ Add Text</button>
      </div>

      <div class="card">
        <h2>Page Options</h2>
        <label>Page Size</label>
        <select v-model="pageSize">
          <option value="A4">A4</option>
          <option value="Letter">Letter</option>
          <option value="Custom">Custom</option>
        </select>
        <div v-if="pageSize === 'Custom'" class="custom-size">
          <input type="number" v-model.number="customSize.width" placeholder="Width (pt)" />
          <input type="number" v-model.number="customSize.height" placeholder="Height (pt)" />
        </div>
        <label>Orientation</label>
        <select v-model="pageOrientation">
          <option value="portrait">Portrait</option>
          <option value="landscape">Landscape</option>
        </select>
      </div>

      <button @click="generatePdf" class="generate-btn">Generate PDF</button>
    </aside>

    <main class="preview-area">
      <h2>Preview</h2>
      <div class="preview-sheet-container">
        <div class="preview-sheet" :style="previewSheetStyle">
          <div 
            class="draggable-qr" 
            :style="{ 
              width: qrSize + 'px', 
              height: qrSize + 'px', 
              top: qrPosition.y + 'px', 
              left: qrPosition.x + 'px' 
            }"
            @mousedown="startDrag($event, 'qr')"
          >
            <QrcodeVue :value="qrValue" :size="qrSize" level="H" />
          </div>
          <div 
            v-for="item in additionalTexts" 
            :key="item.id" 
            class="draggable-text"
            :data-text-id="item.id"
            :style="{
              top: item.position.y + 'px',
              left: item.position.x + 'px',
              fontSize: item.style.fontSize + 'px',
              fontFamily: item.style.fontFamily,
              color: item.style.color,
              fontWeight: item.style.bold ? 'bold' : 'normal',
              fontStyle: item.style.italic ? 'italic' : 'normal',
              textDecoration: item.style.underline ? 'underline' : 'none',
              textAlign: item.style.textAlign
            }"
            @mousedown="startDrag($event, item.id)"
          >
            {{ item.text }}
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
:root {
  --sidebar-width: 420px;
  --border-color: #e0e0e0;
  --bg-light: #f4f5f7;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  margin: 0;
  color: #333;
  background-color: var(--bg-light);
}

#app-container {
  display: flex;
  height: 100vh;
}

.sidebar {
  width: var(--sidebar-width);
  padding: 20px;
  background: #fff;
  border-right: 1px solid var(--border-color);
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 20px;
  box-shadow: 2px 0 5px rgba(0,0,0,0.05);
}

.sidebar h1 {
  font-size: 1.5rem;
  margin: 0;
  text-align: center;
}

.card {
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 15px;
}

.card h2 {
  font-size: 1rem;
  margin-top: 0;
  border-bottom: 1px solid var(--border-color);
  padding-bottom: 10px;
  margin-bottom: 15px;
}

label {
  display: block;
  font-size: 0.875rem;
  margin-bottom: 5px;
  font-weight: 500;
}

input[type="text"],
input[type="number"],
select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  margin-bottom: 10px;
}

.text-config {
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 10px;
  margin-bottom: 15px;
  background: #fafafa;
}

.text-input-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}

.text-formatting {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.format-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.format-row label {
  min-width: 70px;
  margin-bottom: 0;
  font-size: 0.8rem;
}

.format-row input[type="number"],
.format-row select,
.format-row input[type="color"] {
  flex: 1;
  margin-bottom: 0;
  padding: 4px 6px;
  font-size: 0.85rem;
}

.format-row input[type="color"] {
  width: 40px;
  height: 30px;
  padding: 2px;
  border-radius: 4px;
}

.format-buttons {
  display: flex;
  gap: 4px;
  margin-top: 5px;
}

.format-btn {
  width: 30px;
  height: 30px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: #fff;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  transition: all 0.2s;
}

.format-btn:hover {
  background: #f0f0f0;
}

.format-btn.active {
  background: #007bff;
  color: white;
  border-color: #0056b3;
}

.add-btn, .remove-btn {
  border: none;
  cursor: pointer;
  font-weight: bold;
}

.add-btn {
  width: 100%;
  padding: 8px;
  border-radius: 4px;
  background: #eef;
  border: 1px solid #cce;
  color: #337;
}

.remove-btn {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #fee;
  color: red;
  border: 1px solid #fcc;
}

.generate-btn {
  margin-top: auto;
  padding: 12px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.2s;
}
.generate-btn:hover {
  background: #36a372;
}

.preview-area {
  flex-grow: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.preview-area h2 {
    text-align: center;
    margin-top: 0;
}

.preview-sheet-container {
  flex-grow: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: var(--bg-light);
  border-radius: 8px;
}

.preview-sheet {
  /* Styles are now applied dynamically via computed property */
  transform-origin: top left;
}

.draggable-qr, .draggable-text {
  position: absolute;
  cursor: move;
  user-select: none;
  border: 1px dashed transparent;
  transition: border-color 0.2s;
}

.draggable-qr:hover, .draggable-text:hover {
  border-color: #007bff;
}

.draggable-text {
  padding: 5px;
  background-color: rgba(255, 255, 255, 0.8);
  max-width: calc(100% - 40px); /* Leave some margin from the right edge */
  word-wrap: break-word;
  overflow-wrap: break-word;
  white-space: pre-wrap;
  line-height: 1.2;
}
</style>
