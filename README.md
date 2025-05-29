---

## 🌟 Advanced Tabbar Component

### 🛠️ Feature-Rich Tab Management System using Vue.js 3

> **Author:** Aaron Joel B C
> **Role:** Incubation Trainee
> **Division:** Zoho Analytics - Client Framework, Zoho Corporation
> **Year:** 2025

---

### 📌 Component Overview

The **Dynamic Tabbar** is a powerful Vue 3 UI component that replicates modern browser tab behavior and enhances user interaction with:

* 🎯 Drag-and-drop reordering with real-time 3D ghost feedback
* ♿ Full keyboard navigation support
* 📦 Overflow handling via dialog pop-ups
* 🎞️ Smooth animated transitions
* 🧠 Contextual tab typing using icons

![Tabbar Overview](assets/tabbar_overview.png)

---

### 🧩 Core Features

#### 🚀 Drag-and-Drop Implementation

A 3D ghost is generated while dragging to mimic real tab behavior.

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
      rotateX(${rotateX*5}deg)
      rotateY(${rotateY*5}deg)
    `;
  };
}
```

![Drag Preview](assets/drag_preview.png)

---

#### 🧠 Tab Types with Icons

Each tab has a type (`chart`, `excel`, `alert`) and gets an icon using Font Awesome.

```js
propertyIcon: ["fa-pie-chart","fa-table","fa-exclamation-circle"],
computed: {
  propertyNumber() {
    return this.property === 'chart' ? 0 :
           this.property === 'excel'? 1 : 2;
  }
}
```

![Tab Types](assets/tab_types.png)

---

#### 📤 Overflow Management

When tabs overflow the viewport, extra tabs are shifted into a pop-up dialog.

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

![Overflow Dialog](assets/overflow_dialog.png)

---

### ♿ Accessibility Features

#### ⌨️ Full Keyboard Navigation

Navigate tabs without a mouse:

| Key      | Action                  |
| -------- | ----------------------- |
| `←`/`→`  | Move focus between tabs |
| `Home`   | Focus first tab         |
| `End`    | Focus last tab          |
| `Delete` | Close focused tab       |

```js
handleKeydown(e, tabId) {
  switch (e.key) {
    case 'ArrowLeft': this.handleShiftTabKey(e); break;
    case 'ArrowRight': this.handleTabKey(e); break;
    case 'Home': this.focusTab(this.tabs[0].uid); break;
    case 'Delete': this.closeTabAction(tabId); break;
  }
}
```

---

#### 🎯 Focus Restoration

After dialog closes or tabs are removed, focus returns to relevant tab.

```js
cancelDialogBox() {
  if(this.tabs.length) {
    document.querySelector(`.tab-wrapper[data-id="${this.lastFocusedTabId}"]`).focus();
  } else {
    document.getElementById('addTab').focus();
  }
}
```

![Focus Management](assets/focus_management.png)

---

### 🎨 Visual Styling

#### 🌈 Ghost Styling

```css
#drag-ghost {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  filter: drop-shadow(0 0 4px rgba(0, 0, 0, 0.2));
}

#drag-ghost .tab :nth-child(1) {
  color: dodgerblue;
}
```

---

#### 📽️ Animations

```css
@keyframes addDialogueAnimation {
  from { transform: scale(0.5); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.tab-wrapper {
  transition: transform 0.2s ease;
}
```

![Dialog Animation](assets/dialog_animation.png)

---

### ✅ Summary

The Dynamic Tabbar component is:

* 🎯 Highly interactive
* ♿ Accessible
* ⚙️ Configurable
* 📱 Mobile-responsive

#### 🔮 Potential Enhancements

* 💾 Persistent tab state via localStorage
* 🎨 Custom themes and touch support
* 🗂️ Nested tab grouping

---

### 📁 Repository Structure

```
📦 Reactive-Tabs/
 ┣ 📁 components/
 ┃ ┣ 📄 Tabbar.vue
 ┃ ┗ 📄 Tabbar.vue
 ┣ 📁 assets/
 ┣ 📄 README.md
 ┣ 📄 App.vue
 ┗ 📄 main.js
```
