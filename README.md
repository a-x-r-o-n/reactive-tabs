<!-- HEADER BADGES AND BANNER -->

<div align="center">

# 🚀 Vue 3 Dynamic Tabbar

A modern, drag-and-drop-enabled, keyboard-accessible Tabbar component inspired by browser-style tabs — with overflow handling and contextual icons.

![Vue](https://img.shields.io/badge/Vue.js-3.x-brightgreen?logo=vue.js)
![License](https://img.shields.io/badge/license-MIT-blue)
![Accessibility](https://img.shields.io/badge/accessibility-♿-purple)
![Status](https://img.shields.io/badge/project-active-brightgreen)

</div>

---

> 🎓 **Project by Aaron Joel B C**
> 🧪 **Internship:** Zoho Analytics - Client Framework
> 🏢 **Company:** Zoho Corporation
> 📅 **Year:** 2025
> 🌐 **Purpose:** Build an accessible and customizable Tabbar component in Vue 3, emulating browser behavior with enhanced interaction and animation.

---

## 📦 Component Overview

The **Dynamic Tabbar** component provides advanced tab management capabilities:

* 🔀 Drag-and-drop reordering with ghost preview
* ⌨️ Full keyboard navigation support
* 💾 Overflow handling with smart focus restoration
* 🎨 Animated transitions and visual effects
* 🧠 Property-based tab typing (icons for chart, excel, alerts)

---

## ⚙️ Core Features

### 🖱️ Drag-and-Drop Tabs

3D ghost styling for enhanced feedback when dragging:

```js
dragTabAction(event, id) {
  const ghost = dragElement.cloneNode(true);
  ghost.id = 'drag-ghost';
  ghost.style.transform = 'translate(-50%, -50%) perspective(800px)';
  
  const moveGhost = (e) => {
    const rotateX = -deltaY * 0.4;
    const rotateY = deltaX * 0.3;
    ghost.style.transform = `
      translate(-50%, -50%)
      perspective(800px)
      rotateX(${rotateX * 5}deg)
      rotateY(${rotateY * 5}deg)
    `;
  };
}
```

### 🧩 Tab Types with Icons

Property-based rendering with Font Awesome icons:

```js
// In Tab.vue
propertyIcon: ["fa-pie-chart","fa-table","fa-exclamation-circle"],
computed: {
  propertyNumber() {
    return this.property === 'chart' ? 0 : 
           this.property === 'excel' ? 1 : 2;
  }
}
```

---

## 📂 Overflow Management

When tabs exceed visible space, the component pushes extras into an overflow dialog:

```js
toggleTabOverflowDialog() {
  if (this.isOverFlowDialogOpen) {
    document.getElementById("tabOverflowDialog").close();
    this.isOverFlowDialogOpen = false;
    document.querySelector(`.tab-wrapper[data-id="${this.lastFocusedTabId}"]`).focus();
  } else {
    document.getElementById("tabOverflowDialog").show();
    this.isOverFlowDialogOpen = true;
  }
}
```

---

## ♿ Accessibility Features

### ⌨️ Keyboard Navigation

Supports:

* Left/Right arrows to switch tabs
* Home/End to jump
* Delete to remove
* Focus restoration

```js
handleKeydown(e, tabId) {
  switch (e.key) {
    case 'ArrowLeft':
      this.handleShiftTabKey(e); break;
    case 'ArrowRight':
      this.handleTabKey(e); break;
    case 'Home':
      this.focusTab(this.tabs[0].uid); break;
    case 'Delete':
      this.closeTabAction(tabId); break;
  }
}
```

### 🔙 Focus Management

Focus is returned intelligently after tab removal or dialog closure:

```js
cancelDialogBox() {
  if(this.tabs.length) {
    document.querySelector(`.tab-wrapper[data-id="${this.lastFocusedTabId}"]`).focus();
  } else {
    document.getElementById('addTab').focus();
  }
}
```

---

## 🎨 Visual Design

### 🧼 Ghost Styling

```css
#drag-ghost {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  filter: drop-shadow(0 0 4px rgba(0, 0, 0, 0.2));
}

#drag-ghost .tab :nth-child(1) {
  color: dodgerblue;
}
```

### 💫 Animations

Smooth transitions for dialog and tab moves:

```css
@keyframes addDialogueAnimation {
  from { transform: scale(0.5); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.tab-wrapper {
  transition: transform 0.2s ease;
}
```

---

## 🧠 Future Enhancements

* 💾 **Tab persistence** using `localStorage`
* 📌 **Custom tab types** with extended icons
* 📱 **Touch-friendly UX** and gesture support

---

## 📁 Project Structure

```bash
src/
├── components/
│   ├── Tab.vue
│   ├── Tabbar.vue
│   └── OverflowDialog.vue
├── App.vue
└── main.js
```

---

## Project Setup

```sh
npm install
````

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

---

## 📜 License

This project is licensed under the **MIT License**.


