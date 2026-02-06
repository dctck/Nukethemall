import React, { useState, useRef, useEffect } from 'react';

const NukeMemes = () => {
  const [currentImage, setCurrentImage] = useState(null);
  const [nukeCount, setNukeCount] = useState(0);
  const [history, setHistory] = useState([]);
  const [lastEffects, setLastEffects] = useState([]);
  const canvasRef = useRef(null);
  const fileInputRef = useRef(null);

  const loadImage = (file) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        setCurrentImage(img);
        setNukeCount(0);
        setHistory([]);
        setLastEffects([]);
        drawImageToCanvas(img);
      };
      img.src = e.target.result;
    };
    reader.readAsDataURL(file);
  };

  const drawImageToCanvas = (img) => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    canvas.width = img.width;
    canvas.height = img.height;
    ctx.drawImage(img, 0, 0);
  };

  const applyGlitchEffect = (imageData, strength) => {
    const data = imageData.data;
    const offset = Math.floor((strength / 10) * 20) + 5;
    const frequency = 0.7 + (strength / 10) * 0.15;
    
    for (let i = 0; i < data.length; i += 4) {
      if (Math.random() > frequency) {
        const shiftIndex = Math.min(i + offset * 4, data.length - 4);
        [data[i], data[shiftIndex]] = [data[shiftIndex], data[i]];
      }
    }
  };

  const applyRGBShift = (ctx, width, height, strength) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const shift = Math.floor((strength / 10) * 10) + 3;
    
    const tempData = new Uint8ClampedArray(data);
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const i = (y * width + x) * 4;
        const shiftedX = Math.min(x + shift, width - 1);
        const shiftedI = (y * width + shiftedX) * 4;
        
        data[i] = tempData[shiftedI];
        data[i + 2] = tempData[i + 2];
      }
    }
    
    ctx.putImageData(imageData, 0, 0);
  };

  const applyNoise = (imageData, strength) => {
    const data = imageData.data;
    const intensity = 0.05 + (strength / 10) * 0.15;
    const range = (strength / 10) * 80;
    
    for (let i = 0; i < data.length; i += 4) {
      if (Math.random() < intensity) {
        const noise = (Math.random() - 0.5) * range;
        data[i] = Math.max(20, Math.min(235, data[i] + noise));
        data[i + 1] = Math.max(20, Math.min(235, data[i + 1] + noise));
        data[i + 2] = Math.max(20, Math.min(235, data[i + 2] + noise));
      }
    }
  };

  const applyColorDistortion = (imageData, strength = 5) => {
    const data = imageData.data;
    const hueShift = (strength / 10) * 180;
    const satMult = 0.5 + (strength / 10) * 1.5;
    
    for (let i = 0; i < data.length; i += 4) {
      let r = data[i] / 255;
      let g = data[i + 1] / 255;
      let b = data[i + 2] / 255;
      
      const max = Math.max(r, g, b);
      const min = Math.min(r, g, b);
      let h, s, l = (max + min) / 2;
      
      if (max === min) {
        h = s = 0;
      } else {
        const d = max - min;
        s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
        switch (max) {
          case r: h = ((g - b) / d + (g < b ? 6 : 0)) / 6; break;
          case g: h = ((b - r) / d + 2) / 6; break;
          case b: h = ((r - g) / d + 4) / 6; break;
        }
      }
      
      h = (h * 360 + hueShift) % 360 / 360;
      s = Math.min(s * satMult, 1);
      
      const hue2rgb = (p, q, t) => {
        if (t < 0) t += 1;
        if (t > 1) t -= 1;
        if (t < 1/6) return p + (q - p) * 6 * t;
        if (t < 1/2) return q;
        if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
        return p;
      };
      
      if (s === 0) {
        r = g = b = l;
      } else {
        const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
        const p = 2 * l - q;
        r = hue2rgb(p, q, h + 1/3);
        g = hue2rgb(p, q, h);
        b = hue2rgb(p, q, h - 1/3);
      }
      
      data[i] = r * 255;
      data[i + 1] = g * 255;
      data[i + 2] = b * 255;
    }
  };

  const applyPosterize = (imageData, strength = 5) => {
    const data = imageData.data;
    const levels = Math.max(2, Math.floor(11 - (strength / 10) * 8));
    const step = 255 / levels;
    
    for (let i = 0; i < data.length; i += 4) {
      data[i] = Math.floor(data[i] / step) * step;
      data[i + 1] = Math.floor(data[i + 1] / step) * step;
      data[i + 2] = Math.floor(data[i + 2] / step) * step;
    }
  };

  const applyEdgeDetection = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newData = new Uint8ClampedArray(data);
    
    const kernel = [
      [-1, -1, -1],
      [-1,  8, -1],
      [-1, -1, -1]
    ];
    
    for (let y = 1; y < height - 1; y++) {
      for (let x = 1; x < width - 1; x++) {
        let r = 0, g = 0, b = 0;
        
        for (let ky = -1; ky <= 1; ky++) {
          for (let kx = -1; kx <= 1; kx++) {
            const idx = ((y + ky) * width + (x + kx)) * 4;
            const weight = kernel[ky + 1][kx + 1];
            r += data[idx] * weight;
            g += data[idx + 1] * weight;
            b += data[idx + 2] * weight;
          }
        }
        
        const i = (y * width + x) * 4;
        newData[i] = Math.abs(r);
        newData[i + 1] = Math.abs(g);
        newData[i + 2] = Math.abs(b);
      }
    }
    
    ctx.globalAlpha = 0.4;
    const tempCanvas = document.createElement('canvas');
    tempCanvas.width = width;
    tempCanvas.height = height;
    const tempCtx = tempCanvas.getContext('2d');
    const newImageData = tempCtx.createImageData(width, height);
    newImageData.data.set(newData);
    tempCtx.putImageData(newImageData, 0, 0);
    ctx.drawImage(tempCanvas, 0, 0);
    ctx.globalAlpha = 1.0;
  };

  const applyKaleidoscope = (ctx, width, height, strength = 5) => {
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    ctx.globalAlpha = 0.5;
    
    const segments = 2 + Math.floor(Math.random() * 3);
    const centerX = width / 2;
    const centerY = height / 2;
    
    for (let i = 1; i < segments; i++) {
      ctx.save();
      ctx.translate(centerX, centerY);
      ctx.rotate((Math.PI * 2 / segments) * i);
      if (i % 2 === 0) {
        ctx.scale(-1, 1);
      }
      ctx.drawImage(tempCanvas, -centerX, -centerY);
      ctx.restore();
    }
    
    ctx.globalAlpha = 1.0;
  };

  const applyBlockGlitch = (ctx, width, height, strength = 5) => {
    const blockCount = Math.floor((strength / 10) * 30) + 15;
    
    for (let i = 0; i < blockCount; i++) {
      const maxWidth = Math.floor((strength / 10) * 150) + 10;
      const maxHeight = Math.floor((strength / 10) * 80) + 5;
      const blockWidth = Math.floor(Math.random() * maxWidth) + 10;
      const blockHeight = Math.floor(Math.random() * maxHeight) + 5;
      const srcX = Math.floor(Math.random() * Math.max(1, width - blockWidth));
      const srcY = Math.floor(Math.random() * Math.max(1, height - blockHeight));
      const destX = Math.floor(Math.random() * Math.max(1, width - blockWidth));
      const destY = Math.floor(Math.random() * Math.max(1, height - blockHeight));
      
      const blockData = ctx.getImageData(srcX, srcY, blockWidth, blockHeight);
      ctx.putImageData(blockData, destX, destY);
    }
  };

  const applyChromatic = (ctx, width, height, strength) => {
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    const offset = Math.floor((strength / 10) * 15) + 3;
    
    ctx.globalCompositeOperation = 'screen';
    
    ctx.save();
    ctx.globalAlpha = 0.8;
    ctx.filter = 'brightness(1.2) contrast(1.3)';
    ctx.drawImage(tempCanvas, -offset, 0);
    ctx.restore();
    
    ctx.save();
    ctx.globalAlpha = 0.8;
    ctx.filter = 'brightness(0.8) contrast(1.3) hue-rotate(180deg)';
    ctx.drawImage(tempCanvas, offset, 0);
    ctx.restore();
    
    ctx.globalCompositeOperation = 'source-over';
    ctx.globalAlpha = 1.0;
  };

  const applyRadialBlur = (ctx, width, height, strength = 5) => {
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    const centerX = width / 2;
    const centerY = height / 2;
    const blurStrength = (strength / 10) * 0.02 + 0.01; // FIXED: renamed variable
    
    ctx.globalAlpha = 0.3;
    for (let i = 0; i < 8; i++) {
      const scale = 1 + blurStrength * i;
      ctx.save();
      ctx.translate(centerX, centerY);
      ctx.scale(scale, scale);
      ctx.translate(-centerX, -centerY);
      ctx.drawImage(tempCanvas, 0, 0);
      ctx.restore();
    }
    ctx.globalAlpha = 1.0;
  };

  const applySwirl = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const centerX = width / 2;
    const centerY = height / 2;
    const radius = Math.min(width, height) / 2;
    const angle = ((strength / 10) - 0.5) * Math.PI * 2; // Use strength
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const dx = x - centerX;
        const dy = y - centerY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        
        if (distance < radius) {
          const percent = (radius - distance) / radius;
          const rotateAngle = percent * angle;
          
          const srcX = Math.cos(rotateAngle) * dx - Math.sin(rotateAngle) * dy + centerX;
          const srcY = Math.sin(rotateAngle) * dx + Math.cos(rotateAngle) * dy + centerY;
          
          if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
            const srcIdx = (Math.floor(srcY) * width + Math.floor(srcX)) * 4;
            const destIdx = (y * width + x) * 4;
            
            newData[destIdx] = data[srcIdx];
            newData[destIdx + 1] = data[srcIdx + 1];
            newData[destIdx + 2] = data[srcIdx + 2];
            newData[destIdx + 3] = 255;
          }
        } else {
          const srcIdx = (y * width + x) * 4;
          const destIdx = (y * width + x) * 4;
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = 255;
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyGlass = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const scale = Math.floor((strength / 10) * 15) + 5; // Use strength
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const offsetX = Math.floor((Math.random() - 0.5) * scale * 2);
        const offsetY = Math.floor((Math.random() - 0.5) * scale * 2);
        
        const srcX = Math.max(0, Math.min(width - 1, x + offsetX));
        const srcY = Math.max(0, Math.min(height - 1, y + offsetY));
        
        const srcIdx = (srcY * width + srcX) * 4;
        const destIdx = (y * width + x) * 4;
        
        newData[destIdx] = data[srcIdx];
        newData[destIdx + 1] = data[srcIdx + 1];
        newData[destIdx + 2] = data[srcIdx + 2];
        newData[destIdx + 3] = 255;
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyBulge = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const centerX = width / 2;
    const centerY = height / 2;
    const radius = Math.min(width, height) * 0.4;
    const bulgeStrength = (strength / 10) * 0.5 + 0.3; // FIXED: renamed variable
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const dx = x - centerX;
        const dy = y - centerY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        
        if (distance < radius) {
          const percent = distance / radius;
          const bulge = Math.sin(percent * Math.PI / 2);
          const scale = 1 - bulgeStrength * bulge;
          
          const srcX = Math.floor(centerX + dx * scale);
          const srcY = Math.floor(centerY + dy * scale);
          
          if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
            const srcIdx = (srcY * width + srcX) * 4;
            const destIdx = (y * width + x) * 4;
            
            newData[destIdx] = data[srcIdx];
            newData[destIdx + 1] = data[srcIdx + 1];
            newData[destIdx + 2] = data[srcIdx + 2];
            newData[destIdx + 3] = 255;
          }
        } else {
          const srcIdx = (y * width + x) * 4;
          const destIdx = (y * width + x) * 4;
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = 255;
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyPinch = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const centerX = width / 2;
    const centerY = height / 2;
    const radius = Math.min(width, height) * 0.4;
    const pinchStrength = (strength / 10) * 0.5 + 0.3; // FIXED: renamed variable
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const dx = x - centerX;
        const dy = y - centerY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        
        if (distance < radius) {
          const percent = distance / radius;
          const pinch = Math.sin(percent * Math.PI / 2);
          const scale = 1 + pinchStrength * pinch;
          
          const srcX = Math.floor(centerX + dx * scale);
          const srcY = Math.floor(centerY + dy * scale);
          
          if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
            const srcIdx = (srcY * width + srcX) * 4;
            const destIdx = (y * width + x) * 4;
            
            newData[destIdx] = data[srcIdx];
            newData[destIdx + 1] = data[srcIdx + 1];
            newData[destIdx + 2] = data[srcIdx + 2];
            newData[destIdx + 3] = 255;
          }
        } else {
          const srcIdx = (y * width + x) * 4;
          const destIdx = (y * width + x) * 4;
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = 255;
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyCrystallize = (ctx, width, height, strength = 5) => {
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    ctx.clearRect(0, 0, width, height);
    
    const cellSize = Math.floor((strength / 10) * 30) + 10; // Use strength
    
    for (let y = 0; y < height; y += cellSize) {
      for (let x = 0; x < width; x += cellSize) {
        const sampleX = Math.min(x + Math.floor(Math.random() * cellSize), width - 1);
        const sampleY = Math.min(y + Math.floor(Math.random() * cellSize), height - 1);
        
        const imageData = tempCtx.getImageData(sampleX, sampleY, 1, 1);
        const pixel = imageData.data;
        
        ctx.fillStyle = `rgb(${pixel[0]}, ${pixel[1]}, ${pixel[2]})`;
        
        const points = Math.floor(Math.random() * 4) + 4;
        ctx.beginPath();
        for (let i = 0; i < points; i++) {
          const angle = (Math.PI * 2 * i) / points + Math.random() * 0.5;
          const radius = cellSize * (Math.random() * 0.5 + 0.5);
          const px = x + cellSize / 2 + Math.cos(angle) * radius;
          const py = y + cellSize / 2 + Math.sin(angle) * radius;
          if (i === 0) ctx.moveTo(px, py);
          else ctx.lineTo(px, py);
        }
        ctx.closePath();
        ctx.fill();
      }
    }
  };

  const applyEmboss = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newData = new Uint8ClampedArray(data);
    
    const kernel = [
      [-2, -1, 0],
      [-1,  1, 1],
      [ 0,  1, 2]
    ];
    
    for (let y = 1; y < height - 1; y++) {
      for (let x = 1; x < width - 1; x++) {
        let r = 0, g = 0, b = 0;
        
        for (let ky = -1; ky <= 1; ky++) {
          for (let kx = -1; kx <= 1; kx++) {
            const idx = ((y + ky) * width + (x + kx)) * 4;
            const weight = kernel[ky + 1][kx + 1];
            r += data[idx] * weight;
            g += data[idx + 1] * weight;
            b += data[idx + 2] * weight;
          }
        }
        
        const i = (y * width + x) * 4;
        newData[i] = Math.max(20, Math.min(235, r + 128));
        newData[i + 1] = Math.max(20, Math.min(235, g + 128));
        newData[i + 2] = Math.max(20, Math.min(235, b + 128));
      }
    }
    
    const newImageData = ctx.createImageData(width, height);
    newImageData.data.set(newData);
    ctx.putImageData(newImageData, 0, 0);
  };

  const applySolarize = (imageData, strength = 5) => {
    const data = imageData.data;
    const threshold = 255 - (strength / 10) * 155;
    
    for (let i = 0; i < data.length; i += 4) {
      if (data[i] > threshold) data[i] = 255 - data[i];
      if (data[i + 1] > threshold) data[i + 1] = 255 - data[i + 1];
      if (data[i + 2] > threshold) data[i + 2] = 255 - data[i + 2];
    }
  };

  const applyRipple = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const amplitude = (strength / 10) * 60 + 20; // 20-80px (increased from 10-30)
    const frequency = (strength / 10) * 0.02 + 0.01; // 0.01-0.03 (slower ripples)
    const waveType = Math.random() > 0.5;
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        let offsetX, offsetY;
        
        if (waveType) {
          offsetX = Math.sin(y * frequency) * amplitude;
          offsetY = Math.cos(x * frequency) * amplitude;
        } else {
          const distance = Math.sqrt(Math.pow(x - width / 2, 2) + Math.pow(y - height / 2, 2));
          offsetX = Math.sin(distance * frequency) * amplitude;
          offsetY = Math.cos(distance * frequency) * amplitude;
        }
        
        const srcX = Math.floor(x + offsetX);
        const srcY = Math.floor(y + offsetY);
        
        if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
          const srcIdx = (srcY * width + srcX) * 4;
          const destIdx = (y * width + x) * 4;
          
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = 255;
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyBurn = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    
    const centerX = width / 2;
    const centerY = height / 2;
    const maxDist = Math.sqrt(centerX * centerX + centerY * centerY);
    const burnIntensity = (strength / 10) * 0.8; // 0-0.8 based on strength
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const i = (y * width + x) * 4;
        
        // Calculate distance from center (normalized)
        const dx = x - centerX;
        const dy = y - centerY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        const normalizedDist = distance / maxDist;
        
        // Create burn gradient from edges
        const burnAmount = Math.pow(normalizedDist, 1.5) * burnIntensity;
        
        let r = data[i];
        let g = data[i + 1];
        let b = data[i + 2];
        
        // Darken and add orange/brown tones
        r = r * (1 - burnAmount * 0.7) + burnAmount * 120; // Add orange
        g = g * (1 - burnAmount * 0.8) + burnAmount * 40;  // Reduce green
        b = b * (1 - burnAmount * 0.9);                     // Reduce blue heavily
        
        // Add some random "char" spots
        if (Math.random() < burnAmount * 0.3) {
          r *= 0.3;
          g *= 0.2;
          b *= 0.1;
        }
        
        data[i] = Math.max(20, Math.min(235, r));
        data[i + 1] = Math.max(20, Math.min(235, g));
        data[i + 2] = Math.max(20, Math.min(235, b));
      }
    }
    
    ctx.putImageData(imageData, 0, 0);
  };

  const applyExplosion = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const centerX = width / 2;
    const centerY = height / 2;
    const maxDisplacement = (strength / 10) * 25 + 5; // 5-30px based on strength
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        // Calculate direction from center
        const dx = x - centerX;
        const dy = y - centerY;
        const distance = Math.sqrt(dx * dx + dy * dy);
        const angle = Math.atan2(dy, dx);
        
        // Add random angular offset for irregular displacement
        const angleOffset = (Math.random() - 0.5) * Math.PI * 0.3;
        const finalAngle = angle + angleOffset;
        
        // Displacement decreases with distance from center (explosion epicenter)
        const distanceFactor = Math.min(distance / (Math.max(width, height) / 2), 1);
        const displacement = (Math.random() * 0.7 + 0.3) * maxDisplacement * (1 - distanceFactor * 0.3);
        
        // Calculate source position (opposite direction for explosion effect)
        const srcX = Math.floor(x - Math.cos(finalAngle) * displacement);
        const srcY = Math.floor(y - Math.sin(finalAngle) * displacement);
        
        if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
          const srcIdx = (srcY * width + srcX) * 4;
          const destIdx = (y * width + x) * 4;
          
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = 255;
        } else {
          const destIdx = (y * width + x) * 4;
          const srcIdx = (y * width + x) * 4;
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = 255;
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyMosaic = (ctx, width, height, strength = 5) => {
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    ctx.clearRect(0, 0, width, height);
    
    const tiles = Math.floor((strength / 10) * 3) + 2; // Use strength
    const tileWidth = width / tiles;
    const tileHeight = height / tiles;
    
    for (let ty = 0; ty < tiles; ty++) {
      for (let tx = 0; tx < tiles; tx++) {
        const flip = Math.random() > 0.5;
        const rotate = Math.floor(Math.random() * 4) * 90;
        
        ctx.save();
        ctx.translate(tx * tileWidth + tileWidth / 2, ty * tileHeight + tileHeight / 2);
        ctx.rotate((rotate * Math.PI) / 180);
        if (flip) ctx.scale(-1, 1);
        ctx.drawImage(tempCanvas, -tileWidth / 2, -tileHeight / 2, tileWidth, tileHeight);
        ctx.restore();
      }
    }
  };

  const applyDisplacement = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const displacementStrength = Math.floor((strength / 10) * 30) + 10; // FIXED: renamed variable
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const i = (y * width + x) * 4;
        
        const displaceX = ((data[i] / 255) - 0.5) * displacementStrength;
        const displaceY = ((data[i + 1] / 255) - 0.5) * displacementStrength;
        
        const srcX = Math.floor(x + displaceX);
        const srcY = Math.floor(y + displaceY);
        
        if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
          const srcIdx = (srcY * width + srcX) * 4;
          newData[i] = data[srcIdx];
          newData[i + 1] = data[srcIdx + 1];
          newData[i + 2] = data[srcIdx + 2];
          newData[i + 3] = 255;
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyChunkyBlocks = (ctx, width, height, strength = 5) => {
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    const chunkCount = Math.floor((strength / 10) * 50) + 20; // Use strength
    
    for (let i = 0; i < chunkCount; i++) {
      const chunkWidth = Math.floor((strength / 10) * 120) + 5;
      const chunkHeight = Math.floor((strength / 10) * 100) + 5;
      
      const srcX = Math.floor(Math.random() * Math.max(1, width - chunkWidth));
      const srcY = Math.floor(Math.random() * Math.max(1, height - chunkHeight));
      
      const destX = Math.floor(Math.random() * width);
      const destY = Math.floor(Math.random() * height);
      
      ctx.globalAlpha = Math.random() * 0.5 + 0.5;
      ctx.drawImage(tempCanvas, srcX, srcY, chunkWidth, chunkHeight, destX, destY, chunkWidth, chunkHeight);
    }
    ctx.globalAlpha = 1.0;
  };

  const applyParticles = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    
    const particleCount = Math.floor((strength / 10) * 800) + 300; // Use strength
    
    for (let i = 0; i < particleCount; i++) {
      const x = Math.floor(Math.random() * width);
      const y = Math.floor(Math.random() * height);
      
      const sizeType = Math.random();
      let size;
      if (sizeType < 0.5) {
        size = Math.floor((strength / 10) * 3) + 1;
      } else if (sizeType < 0.8) {
        size = Math.floor((strength / 10) * 10) + 2;
      } else {
        size = Math.floor((strength / 10) * 20) + 5;
      }
      
      const sampleX = Math.floor(x + (Math.random() - 0.5) * 50);
      const sampleY = Math.floor(y + (Math.random() - 0.5) * 50);
      
      if (sampleX >= 0 && sampleX < width && sampleY >= 0 && sampleY < height) {
        const sampleIdx = (sampleY * width + sampleX) * 4;
        ctx.fillStyle = `rgba(${data[sampleIdx]}, ${data[sampleIdx + 1]}, ${data[sampleIdx + 2]}, ${Math.random() * 0.5 + 0.5})`;
        
        if (Math.random() > 0.7) {
          ctx.fillRect(x, y, size, size * (Math.random() * 2 + 0.5));
        } else {
          ctx.fillRect(x, y, size, size);
        }
      }
    }
  };

  const applySliceShift = (ctx, width, height, strength = 5) => {
    const sliceCount = Math.floor((strength / 10) * 20) + 10; // Use strength
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    for (let i = 0; i < sliceCount; i++) {
      const sliceHeight = Math.floor((strength / 10) * 30) + 3;
      const y = Math.floor(Math.random() * Math.max(1, height - sliceHeight));
      const shift = (Math.random() - 0.5) * width * ((strength / 10) * 0.3);
      
      const sliceData = tempCtx.getImageData(0, y, width, sliceHeight);
      ctx.putImageData(sliceData, shift, y);
    }
  };

  const applyLiquify = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const waveCount = Math.floor((strength / 10) * 3) + 2; // Use strength
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        let offsetX = 0;
        let offsetY = 0;
        
        for (let w = 0; w < waveCount; w++) {
          const freq = (strength / 10) * 0.05 + 0.01;
          const amp = (strength / 10) * 20 + 5;
          offsetX += Math.sin((y + w * 100) * freq) * amp;
          offsetY += Math.cos((x + w * 100) * freq) * amp;
        }
        
        const srcX = Math.floor(x + offsetX);
        const srcY = Math.floor(y + offsetY);
        
        if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
          const srcIdx = (srcY * width + srcX) * 4;
          const destIdx = (y * width + x) * 4;
          
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = 255;
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyWaveDistortion = (ctx, width, height, strength = 5) => {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    const newImageData = ctx.createImageData(width, height);
    const newData = newImageData.data;
    
    const amplitude = (strength / 10) * 60 + 20; // 20-80px (increased from 5-20)
    const frequency = (strength / 10) * 0.02 + 0.005; // 0.005-0.025 (slower waves)
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const offsetX = Math.sin(y * frequency) * amplitude;
        const offsetY = Math.cos(x * frequency) * amplitude;
        
        const srcX = Math.floor(x + offsetX);
        const srcY = Math.floor(y + offsetY);
        
        if (srcX >= 0 && srcX < width && srcY >= 0 && srcY < height) {
          const srcIdx = (srcY * width + srcX) * 4;
          const destIdx = (y * width + x) * 4;
          
          newData[destIdx] = data[srcIdx];
          newData[destIdx + 1] = data[srcIdx + 1];
          newData[destIdx + 2] = data[srcIdx + 2];
          newData[destIdx + 3] = data[srcIdx + 3];
        }
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  };

  const applyDeepFry = (imageData, strength = 5) => {
    const data = imageData.data;
    const contrast = 1 + (strength / 10) * 0.3;
    const saturation = 1 + (strength / 10) * 0.8;
    
    for (let i = 0; i < data.length; i += 4) {
      let r = data[i];
      let g = data[i + 1];
      let b = data[i + 2];
      
      r = ((r / 255 - 0.5) * contrast + 0.5) * 255;
      g = ((g / 255 - 0.5) * contrast + 0.5) * 255;
      b = ((b / 255 - 0.5) * contrast + 0.5) * 255;
      
      const gray = 0.299 * r + 0.587 * g + 0.114 * b;
      r = gray + (r - gray) * saturation;
      g = gray + (g - gray) * saturation;
      b = gray + (b - gray) * saturation;
      
      data[i] = Math.max(15, Math.min(240, r));
      data[i + 1] = Math.max(15, Math.min(240, g));
      data[i + 2] = Math.max(15, Math.min(240, b));
    }
  };

  const applyChannelSwap = (imageData, strength = 5) => {
    const data = imageData.data;
    const swapType = Math.floor((strength / 10) * 6);
    
    for (let i = 0; i < data.length; i += 4) {
      const r = data[i];
      const g = data[i + 1];
      const b = data[i + 2];
      
      switch (swapType) {
        case 0:
          data[i] = g; data[i + 1] = b; data[i + 2] = r;
          break;
        case 1:
          data[i] = b; data[i + 1] = r; data[i + 2] = g;
          break;
        case 2:
          data[i] = r; data[i + 1] = b; data[i + 2] = g;
          break;
        case 3:
          data[i] = g; data[i + 1] = r; data[i + 2] = b;
          break;
        case 4:
          data[i] = b; data[i + 1] = g; data[i + 2] = r;
          break;
        default:
          break;
      }
    }
  };

  const applyJPEGCorruption = (ctx, width, height) => {
    const quality = Math.random() * 0.2 + 0.2;
    const tempCanvas = document.createElement('canvas');
    tempCanvas.width = width;
    tempCanvas.height = height;
    const tempCtx = tempCanvas.getContext('2d');
    tempCtx.drawImage(ctx.canvas, 0, 0);
    
    const dataURL = tempCanvas.toDataURL('image/jpeg', quality);
    const img = new Image();
    img.onload = () => {
      ctx.drawImage(img, 0, 0);
    };
    img.src = dataURL;
  };

  const addOverlay = () => {
    if (!currentImage) return;
    
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;
    
    const currentState = canvas.toDataURL();
    setHistory(prev => [...prev, currentState]);
    
    const scaledWidth = width * 0.7;
    const scaledHeight = height * 0.7;
    
    const x = Math.random() * (width - scaledWidth);
    const y = Math.random() * (height - scaledHeight);
    
    const rotation = (Math.random() - 0.5) * Math.PI;
    
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(canvas, 0, 0);
    
    ctx.save();
    ctx.translate(x + scaledWidth / 2, y + scaledHeight / 2);
    ctx.rotate(rotation);
    ctx.globalAlpha = 0.5;
    ctx.drawImage(tempCanvas, -scaledWidth / 2, -scaledHeight / 2, scaledWidth, scaledHeight);
    ctx.restore();
    
    setLastEffects(['Add Overlay (70% size, 50% opacity)']);
    setNukeCount(prev => prev + 1);
  };

  const applyPattern = () => {
    if (!currentImage) return;
    
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;
    
    const currentState = canvas.toDataURL();
    setHistory(prev => [...prev, currentState]);
    
    // Randomly choose a pattern type
    const patternType = Math.floor(Math.random() * 4);
    const patternNames = ['Pattern Tiling', 'Radial Duplication', 'Transform Repeat', 'Step-and-Repeat'];
    
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    tempCanvas.width = width;
    tempCanvas.height = height;
    tempCtx.drawImage(canvas, 0, 0);
    
    // Clear canvas and create clipping region to prevent drawing outside bounds
    ctx.clearRect(0, 0, width, height);
    ctx.save();
    ctx.beginPath();
    ctx.rect(0, 0, width, height);
    ctx.clip();
    
    // Use center 80% of image to avoid edge artifacts
    const cropMargin = 0.1; // 10% margin on each side
    const cropX = width * cropMargin;
    const cropY = height * cropMargin;
    const cropWidth = width * (1 - 2 * cropMargin);
    const cropHeight = height * (1 - 2 * cropMargin);
    
    switch(patternType) {
      case 0: // Pattern Tiling - Grid duplication
        {
          const cols = Math.floor(Math.random() * 2) + 2; // 2-3 columns
          const rows = Math.floor(Math.random() * 2) + 2; // 2-3 rows
          const tileWidth = width / cols;
          const tileHeight = height / rows;
          
          for (let row = 0; row < rows; row++) {
            for (let col = 0; col < cols; col++) {
              // Draw from center crop area
              ctx.drawImage(tempCanvas, 
                cropX, cropY, cropWidth, cropHeight, // Source: center crop
                col * tileWidth, row * tileHeight, tileWidth, tileHeight); // Destination: grid
            }
          }
        }
        break;
        
      case 1: // Radial Duplication - Around center point
        {
          const copies = Math.floor(Math.random() * 4) + 6; // 6-9 copies
          const centerX = width / 2;
          const centerY = height / 2;
          const scale = 0.4 + Math.random() * 0.2; // 0.4-0.6 scale
          
          ctx.globalAlpha = 0.6;
          for (let i = 0; i < copies; i++) {
            const angle = (Math.PI * 2 * i) / copies;
            ctx.save();
            ctx.translate(centerX, centerY);
            ctx.rotate(angle);
            ctx.scale(scale, scale);
            // Draw center crop only
            ctx.drawImage(tempCanvas, 
              cropX, cropY, cropWidth, cropHeight, // Source: center crop
              -cropWidth * scale / 2, -cropHeight * scale / 2, cropWidth * scale, cropHeight * scale);
            ctx.restore();
          }
          ctx.globalAlpha = 1.0;
        }
        break;
        
      case 2: // Transform Repeat - Repeated scale/rotate/move
        {
          const steps = Math.floor(Math.random() * 3) + 4; // 4-6 steps
          const scaleStep = 0.85 + Math.random() * 0.1; // 0.85-0.95 per step
          const rotateStep = (Math.random() - 0.5) * 0.3; // Random rotation per step
          const moveX = (Math.random() - 0.5) * width * 0.08;
          const moveY = (Math.random() - 0.5) * height * 0.08;
          
          ctx.globalAlpha = 0.7;
          for (let i = 0; i < steps; i++) {
            const currentScale = Math.pow(scaleStep, i);
            const currentRotation = rotateStep * i;
            const currentX = moveX * i;
            const currentY = moveY * i;
            
            ctx.save();
            ctx.translate(width / 2 + currentX, height / 2 + currentY);
            ctx.rotate(currentRotation);
            ctx.scale(currentScale, currentScale);
            // Draw center crop only
            ctx.drawImage(tempCanvas, 
              cropX, cropY, cropWidth, cropHeight, // Source: center crop
              -cropWidth / 2, -cropHeight / 2, cropWidth, cropHeight);
            ctx.restore();
          }
          ctx.globalAlpha = 1.0;
        }
        break;
        
      case 3: // Step-and-Repeat - Set intervals
        {
          const horizontal = Math.random() > 0.5;
          const steps = Math.floor(Math.random() * 2) + 3; // 3-4 steps
          const scale = 0.5 + Math.random() * 0.2; // 0.5-0.7 scale
          const spacing = horizontal ? width / steps : height / steps;
          
          ctx.globalAlpha = 0.65;
          for (let i = 0; i < steps; i++) {
            const x = horizontal ? spacing * i + spacing / 2 : width / 2;
            const y = horizontal ? height / 2 : spacing * i + spacing / 2;
            
            ctx.save();
            ctx.translate(x, y);
            ctx.scale(scale, scale);
            // Draw center crop only
            ctx.drawImage(tempCanvas, 
              cropX, cropY, cropWidth, cropHeight, // Source: center crop
              -cropWidth / 2, -cropHeight / 2, cropWidth, cropHeight);
            ctx.restore();
          }
          ctx.globalAlpha = 1.0;
        }
        break;
    }
    
    ctx.restore(); // Remove clipping
    setLastEffects([patternNames[patternType]]);
    setNukeCount(prev => prev + 1);
  };

  const nukeImage = () => {
    if (!currentImage) return;
    
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;
    
    const currentState = canvas.toDataURL();
    setHistory(prev => [...prev, currentState]);
    
    const strength = Math.floor(Math.random() * 10) + 1;
    
    const numEffects = Math.floor(Math.random() * 2) + 1;
    const allEffects = [
      { fn: () => applyRGBShift(ctx, width, height, strength), name: 'RGB Shift' },
      { fn: () => {
        const imageData = ctx.getImageData(0, 0, width, height);
        applyGlitchEffect(imageData, strength);
        ctx.putImageData(imageData, 0, 0);
      }, name: 'Glitch Effect' },
      { fn: () => {
        const imageData = ctx.getImageData(0, 0, width, height);
        applyNoise(imageData, strength);
        ctx.putImageData(imageData, 0, 0);
      }, name: 'Noise' },
      { fn: () => {
        const imageData = ctx.getImageData(0, 0, width, height);
        applyColorDistortion(imageData, strength);
        ctx.putImageData(imageData, 0, 0);
      }, name: 'Color Distortion' },
      { fn: () => {
        const imageData = ctx.getImageData(0, 0, width, height);
        applyPosterize(imageData, strength);
        ctx.putImageData(imageData, 0, 0);
      }, name: 'Posterize' },
      { fn: () => applyEdgeDetection(ctx, width, height, strength), name: 'Edge Detection' },
      { fn: () => applyKaleidoscope(ctx, width, height, strength), name: 'Kaleidoscope' },
      { fn: () => applyWaveDistortion(ctx, width, height, strength), name: 'Wave Distortion' },
      { fn: () => {
        const imageData = ctx.getImageData(0, 0, width, height);
        applyDeepFry(imageData, strength);
        ctx.putImageData(imageData, 0, 0);
      }, name: 'Deep Fry' },
      { fn: () => {
        const imageData = ctx.getImageData(0, 0, width, height);
        applyChannelSwap(imageData, strength);
        ctx.putImageData(imageData, 0, 0);
      }, name: 'Channel Swap' },
      { fn: () => applyBlockGlitch(ctx, width, height, strength), name: 'Block Glitch' },
      { fn: () => applyChromatic(ctx, width, height, strength), name: 'Chromatic Aberration' },
      { fn: () => applyRadialBlur(ctx, width, height, strength), name: 'Radial Blur' },
      { fn: () => applySwirl(ctx, width, height, strength), name: 'Swirl' },
      { fn: () => applyBurn(ctx, width, height, strength), name: 'Burn' },
      { fn: () => applyExplosion(ctx, width, height, strength), name: 'Explosion' },
      { fn: () => applyDisplacement(ctx, width, height, strength), name: 'Displacement' },
      { fn: () => applyParticles(ctx, width, height, strength), name: 'Particles' },
      { fn: () => applyLiquify(ctx, width, height, strength), name: 'Liquify' },
      { fn: () => applyChunkyBlocks(ctx, width, height, strength), name: 'Chunky Blocks' },
      { fn: () => applySliceShift(ctx, width, height, strength), name: 'Slice Shift' },
      { fn: () => applyGlass(ctx, width, height, strength), name: 'Glass' },
      { fn: () => applyBulge(ctx, width, height, strength), name: 'Bulge' },
      { fn: () => applyPinch(ctx, width, height, strength), name: 'Pinch' },
      { fn: () => applyCrystallize(ctx, width, height, strength), name: 'Crystallize' },
      { fn: () => applyEmboss(ctx, width, height, strength), name: 'Emboss' },
      { fn: () => {
        const imageData = ctx.getImageData(0, 0, width, height);
        applySolarize(imageData, strength);
        ctx.putImageData(imageData, 0, 0);
      }, name: 'Solarize' },
      { fn: () => applyRipple(ctx, width, height, strength), name: 'Ripple' },
    ];
    
    const shuffled = allEffects.sort(() => Math.random() - 0.5);
    const selectedEffects = shuffled.slice(0, numEffects);
    
    // Apply selected effects
    selectedEffects.forEach(effect => effect.fn());
    
    // Store effect names with strength
    const effectNames = selectedEffects.map(e => `${e.name} (${strength})`);
    
    // Rarely add JPEG corruption
    if (Math.random() > 0.85) {
      applyJPEGCorruption(ctx, width, height);
      effectNames.push('JPEG Corruption');
    }
    
    setLastEffects(effectNames);
    setNukeCount(prev => prev + 1);
  };

  const undoNuke = () => {
    if (history.length === 0) return;
    
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    
    const previousState = history[history.length - 1];
    
    const img = new Image();
    img.onload = () => {
      ctx.drawImage(img, 0, 0);
      setHistory(prev => prev.slice(0, -1));
      setNukeCount(prev => Math.max(0, prev - 1));
      setLastEffects([]);
    };
    img.src = previousState;
  };

  const exportImage = () => {
    if (!canvasRef.current) return;
    
    const canvas = canvasRef.current;
    const link = document.createElement('a');
    link.download = `nuked-meme-${nukeCount}.png`;
    link.href = canvas.toDataURL('image/png');
    link.click();
  };

  const resetImage = () => {
    fileInputRef.current.click();
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-purple-900 via-pink-900 to-red-900 p-8">
      <div className="max-w-4xl mx-auto">
        <div className="text-center mb-8">
          <h1 className="text-6xl font-black text-white mb-2 drop-shadow-lg" style={{
            textShadow: '0 0 20px rgba(255,0,255,0.8), 0 0 40px rgba(255,0,0,0.6)'
          }}>
            🔥 NUKE MEMES 🔥
          </h1>
          <p className="text-xl text-pink-200">Deep fry your memes into oblivion</p>
          {nukeCount > 0 && (
            <div className="mt-4 inline-block bg-yellow-400 text-black px-6 py-2 rounded-full font-bold text-lg">
              💥 NUKES: {nukeCount} 💥
            </div>
          )}
        </div>

        <input
          ref={fileInputRef}
          type="file"
          accept="image/*"
          onChange={(e) => e.target.files[0] && loadImage(e.target.files[0])}
          className="hidden"
        />

        {!currentImage ? (
          <div
            onClick={() => fileInputRef.current.click()}
            className="border-4 border-dashed border-pink-400 rounded-lg p-16 text-center cursor-pointer hover:border-pink-300 hover:bg-pink-900/20 transition-all"
          >
            <div className="text-6xl mb-4">📤</div>
            <p className="text-2xl text-white font-bold mb-2">Click to Upload Image</p>
            <p className="text-pink-300">Drop your meme here to start nuking</p>
          </div>
        ) : (
          <>
            <div className="bg-black rounded-lg p-4 mb-6 shadow-2xl">
              <canvas
                ref={canvasRef}
                className="max-w-full mx-auto"
                style={{ imageRendering: 'pixelated' }}
              />
            </div>

            {lastEffects.length > 0 && (
              <div className="mb-6 text-center">
                <div className="inline-block bg-gradient-to-r from-cyan-500 to-blue-500 rounded-lg px-6 py-3 shadow-lg">
                  <p className="text-white font-bold text-lg mb-1">⚡ Last Applied Effects:</p>
                  <div className="flex flex-wrap gap-2 justify-center">
                    {lastEffects.map((effect, index) => (
                      <span key={index} className="bg-white/20 backdrop-blur-sm px-3 py-1 rounded-full text-white text-sm font-semibold">
                        {effect}
                      </span>
                    ))}
                  </div>
                </div>
              </div>
            )}

            <div className="flex gap-4 flex-wrap justify-center">
              <button
                onClick={nukeImage}
                className="px-8 py-4 bg-gradient-to-r from-red-600 to-orange-600 text-white text-2xl font-black rounded-lg hover:from-red-500 hover:to-orange-500 transform hover:scale-105 transition-all shadow-lg"
                style={{
                  textShadow: '2px 2px 4px rgba(0,0,0,0.8)'
                }}
              >
                💣 NUKE THIS 💣
              </button>

              <button
                onClick={addOverlay}
                className="px-8 py-4 bg-gradient-to-r from-purple-600 to-pink-600 text-white text-xl font-bold rounded-lg hover:from-purple-500 hover:to-pink-500 transform hover:scale-105 transition-all shadow-lg"
              >
                🎭 Add Overlay
              </button>

              <button
                onClick={applyPattern}
                className="px-8 py-4 bg-gradient-to-r from-indigo-600 to-purple-600 text-white text-xl font-bold rounded-lg hover:from-indigo-500 hover:to-purple-500 transform hover:scale-105 transition-all shadow-lg"
              >
                🔲 Pattern
              </button>

              <button
                onClick={undoNuke}
                disabled={history.length === 0}
                className={`px-8 py-4 text-white text-xl font-bold rounded-lg transform hover:scale-105 transition-all shadow-lg ${
                  history.length === 0 
                    ? 'bg-gray-600 cursor-not-allowed opacity-50' 
                    : 'bg-gradient-to-r from-yellow-600 to-amber-600 hover:from-yellow-500 hover:to-amber-500'
                }`}
              >
                ↩️ Undo
              </button>

              <button
                onClick={exportImage}
                className="px-8 py-4 bg-gradient-to-r from-green-600 to-teal-600 text-white text-xl font-bold rounded-lg hover:from-green-500 hover:to-teal-500 transform hover:scale-105 transition-all shadow-lg"
              >
                💾 Export PNG
              </button>

              <button
                onClick={resetImage}
                className="px-8 py-4 bg-gradient-to-r from-blue-600 to-cyan-600 text-white text-xl font-bold rounded-lg hover:from-blue-500 hover:to-cyan-500 transform hover:scale-105 transition-all shadow-lg"
              >
                🔄 New Image
              </button>
            </div>

            <div className="mt-6 text-center text-pink-200">
              <p className="text-sm">Right-click the image to copy • Keep nuking for maximum destruction</p>
            </div>
          </>
        )}

        <div className="mt-12 text-center text-pink-300 text-sm">
          <p>Inspired by r/NukedMemes • Crank everything to 11 🎸</p>
        </div>
      </div>
    </div>
  );
};

export default NukeMemes;
