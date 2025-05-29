<script>
import Tab from './Tab.vue';
let uniqueId = 0;

export default {
    name: 'Tabbar',
    components: { Tab },
    data() {
        return {
            tabs: [],
            draggedTabId: null,
            isFirstDragged: false,
            activeTab: null,
            liveAnnouncement: '',
            lastKeyboardTabFocus: null,
            curretHoveredElement: null,
        };
    },
    methods: {
        addTabAction() {
            const newTab = {
                uid: uniqueId++,
                content: 'Tab ' + uniqueId
            };
            this.tabs.push(newTab);
            this.activeTab = newTab.uid;
            this.liveAnnouncement = `Added tab ${newTab.content}`;
                this.$nextTick(() => {
        const tabEl = document.querySelector(`.tab-wrapper[data-id="${newTab.uid}"]`);
        if (tabEl) {
            tabEl.focus();
        }
    });
        },
        closeTabAction(id) {
            const tab = this.tabs.find(t => t.uid === id);
            this.tabs = this.tabs.filter(tab => tab.uid !== id);
            this.liveAnnouncement = `Closed tab ${tab.content}`;
            
            if (this.activeTab === id) {
                this.activeTab = this.tabs.length > 0 ? this.tabs[this.tabs.length - 1].uid : null;
            }
        },
        dragTabAction(event, id) {

            /*-------------------------------------*/
            this.activeTab = id;
            const tabElement = document.querySelector(`.tab-wrapper[data-id="${id}"]`);
            tabElement.classList.add('hovered');

            /*--------------------------------------*/


            this.draggedTabId = id;
            event.dataTransfer.effectAllowed = 'move';
            event.dataTransfer.setData('text/plain', id.toString());
            
            const dragElement = event.target.closest('.tab-wrapper');
            dragElement.classList.add('dragging');
            
            // Create ghost element
            const ghost = dragElement.cloneNode(true);
            ghost.id = 'drag-ghost';
            ghost.style.position = 'fixed';
            ghost.style.backgroundColor = '#ffffff';
            ghost.style.pointerEvents = 'none';
            ghost.style.zIndex = '9999';
            ghost.style.opacity = '1';
            ghost.style.transform = 'translate(-50%, -50%)';
            ghost.style.transition = 'none';
            ghost.style.width = `${dragElement.offsetWidth}px`;
            
            // Position initially
            const rect = dragElement.getBoundingClientRect();
            ghost.style.left = `${rect.left + rect.width/2}px`;
            ghost.style.top = `${rect.top + rect.height/2}px`;
            
            document.body.appendChild(ghost);
            
            // Update position during drag
            const moveGhost = (e) => {
                requestAnimationFrame(() => {
                    ghost.style.left = `${e.clientX}px`;
                    ghost.style.top = `${e.clientY}px`;
                });
            };
            
            // Clean up
            const cleanUp = () => {
                document.removeEventListener('dragover', moveGhost);
                if (ghost.parentNode) {
                    document.body.removeChild(ghost);
                }
                dragElement.classList.remove('dragging');
            };
            
            document.addEventListener('dragover', moveGhost);
            document.addEventListener('dragend', cleanUp, { once: true });
            
            event.dataTransfer.setDragImage(new Image(), 0, 0);
            this.liveAnnouncement = `Dragging tab`;
        },
        dragEndAction(event) {
            /*-------------------*/
            document.querySelectorAll('.tab-wrapper.hovered').forEach(tab => {
            if (tab !== this.currentHoveringElement) {
                tab.classList.remove('hovered');
              }
            });
            
            const firstHoveredTab = document.querySelector('.tab-wrapper.hovered');
            if (firstHoveredTab && firstHoveredTab !== this.currentHoveringElement) {
              firstHoveredTab.classList.remove('hovered');
            }
            /*-------------------*/

            
            event.target.closest('.tab-wrapper').classList.remove('dragging');
            this.draggedTabId = null;

        

            /*-------------------*/
            const tabElement = document.querySelector(`.tab-wrapper[data-id="${this.activeTab}"]`);
            if (tabElement) {
                tabElement.focus();
            }
            /*-------------------*/

        },
        handleDragOverTab(event, hoveredTabId) {
            event.preventDefault();

            
            

            if (this.draggedTabId === null || this.draggedTabId === hoveredTabId) return;

            const draggedIndex = this.tabs.findIndex(tab => tab.uid === this.draggedTabId);
            const hoveredIndex = this.tabs.findIndex(tab => tab.uid === hoveredTabId);
            if (draggedIndex === hoveredIndex) return;

            const hoveredElement = event.currentTarget;


            /*--------------------------------------------------------*/
            hoveredElement.classList.add('hovered');
            this.curretHoveredElement = hoveredElement;
            /*--------------------------------------------------------*/

            
            const hoveredRect = hoveredElement.getBoundingClientRect();
            const draggedElement = document.querySelector(`.tab-wrapper[data-id="${this.draggedTabId}"]`);
            if (!draggedElement) return;

            const draggedRect = draggedElement.getBoundingClientRect();
            const draggedLeft = draggedRect.left;
            const draggedRight = draggedRect.right;
            const hoveredMidpoint = hoveredRect.right - (hoveredRect.width / 2.0);
            let newIndex;
            const buffer = draggedRect.width * 0.25;
            const isRTL = document.dir === 'rtl' || document.body.dir === 'rtl';

            if (isRTL) {
                if (draggedRight < (hoveredMidpoint - buffer)) {
                    newIndex = hoveredIndex;
                } else if (draggedLeft > (hoveredMidpoint + buffer)) {
                    newIndex = hoveredIndex + 1;
                } else {
                    this.isFirstDragged = true;
                    newIndex = hoveredIndex;
                    return;
                }
            }
            else {
                if (draggedRight < (hoveredMidpoint - buffer)) {
                    newIndex = hoveredIndex + 1;
                } else if (draggedLeft > (hoveredMidpoint + buffer)) {
                    newIndex = hoveredIndex;
                } else {
                    this.isFirstDragged = true;
                    newIndex = hoveredIndex;
                    return;
                }
            }

            if (newIndex === draggedIndex || newIndex === (draggedIndex + 1)) return;

            if (this.isFirstDragged) {
                this.isFirstDragged = false;
            } else {
                const [draggedTab] = this.tabs.splice(draggedIndex, 1);
                this.tabs.splice(newIndex > draggedIndex ? newIndex - 1 : newIndex, 0, draggedTab);
            }
        },
        handleTabKey(e) {
            if (this.tabs.length === 0) {
                document.getElementById('addTab').focus();
                return;
            }
            const currentIndex = this.tabs.findIndex(tab => tab.uid === this.activeTab);
            
            // If we're on the last tab, move focus to add button
            if (currentIndex === this.tabs.length - 1) {
                e.preventDefault();
                document.getElementById('addTab').focus();
            } else {
                const nextIndex = (currentIndex + 1) % this.tabs.length;
                this.focusTab(this.tabs[nextIndex].uid);
            }
        },
        handleShiftTabKey(e) {
    if (this.tabs.length === 0) {
        document.getElementById('addTab').focus();
        return;
    }
    const currentIndex = this.tabs.findIndex(tab => tab.uid === this.activeTab);

    // Move focus to the previous tab
    const prevIndex = (currentIndex - 1 + this.tabs.length) % this.tabs.length;
    this.focusTab(this.tabs[prevIndex].uid);
},
        handleTabOut(e) {
            // When tabbing out from add button, return to first tab if exists
            if (this.tabs.length > 0) {
                e.preventDefault();
                this.focusTab(this.tabs[0].uid);
            }
        },
        handleShiftTabIn(e) {
            // When shift+tabbing into add button from below, focus last tab
            if (this.tabs.length > 0) {
                e.preventDefault();
                this.focusTab(this.tabs[this.tabs.length - 1].uid);
            }
        },
        handleKeydown(e, tabId) {
            const tabCount = this.tabs.length;
            const currentIndex = this.tabs.findIndex(tab => tab.uid === tabId);

            switch(e.key) {
                case 'ArrowLeft':
                    e.preventDefault();
                    this.handleShiftTabKey(e);
                    break;
                case 'ArrowRight':
                    e.preventDefault();
                    this.handleTabKey(e);
                    break;
                case 'Home':
                    e.preventDefault();
                    this.focusTab(this.tabs[0].uid);
                    break;
                case 'End':
                    e.preventDefault();
                    this.focusTab(this.tabs[tabCount - 1].uid);
                    break;
                case 'Enter':
                case ' ':
                    e.preventDefault();
                    this.activeTab = tabId;
                    break;
                case 'Delete':
                    this.closeTabAction(tabId);
                    if (tabCount > 1) {
                        const newIndex = currentIndex === tabCount - 1 ? currentIndex - 1 : currentIndex;
                        this.$nextTick(() => this.focusTab(this.tabs[newIndex].uid));
                    } else {
                        this.$nextTick(() => document.getElementById('addTab').focus());
                    }
                    break;
            }
        },
        focusTab(tabId) {
            if (tabId === 'addTab') {
                this.$nextTick(() => {
                    document.getElementById('addTab').focus();
                });
            } else {
                this.activeTab = tabId;
                this.$nextTick(() => {
                    const tabEl = document.querySelector(`.tab-wrapper[data-id="${tabId}"]`);
                    if (tabEl) {
                        tabEl.focus();
                        tabEl.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
                    }
                });
            }
        }
    }
};
</script>

<template>
    <nav aria-label="Tab navigation">
        <div class="tabbar">
            <transition-group 
                name="reorder" 
                tag="div" 
                class="tabs-wrapper" 
                role="tablist"
                @keydown.tab.prevent="handleTabKey"
                @keydown.shift.tab.prevent="handleShiftTabKey"
            >
                <div 
                    v-for="tab in tabs" 
                    :key="tab.uid" 
                    class="tab-wrapper" 
                    draggable="true"
                    :data-id="tab.uid"
                    role="tab"
                    :aria-selected="activeTab === tab.uid"
                    :tabindex="activeTab === tab.uid ? 0 : -1"
                    @dragstart="(e) => dragTabAction(e, tab.uid)" 
                    @dragend="dragEndAction"
                    @dragover.prevent="(e) => handleDragOverTab(e, tab.uid)"
                    @keydown="(e) => handleKeydown(e, tab.uid)"
                    @click="focusTab(tab.uid)"
                >
                    <Tab :id="tab.uid" :content="tab.content" @close="closeTabAction" />
                </div>
            </transition-group>
            <button 
                id="addTab" 
                aria-label="Add new tab" 
                @click="addTabAction"
                @keydown.tab="handleTabOut"
                @keydown.shift.tab="handleShiftTabIn"
                tabindex="0"
            >+</button>
        </div>
        <div class="sr-only">{{ liveAnnouncement }}</div>
    </nav>
</template>

<style scoped>
.tab-wrapper {
    user-select: none;
    outline: none;
    position: relative;
}
/* .tab-wrapper:focus-visible {
    outline: 2px solid #0066cc;
    outline-offset: 2px;
    z-index: 1;
} */
#drag-ghost {
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    filter: drop-shadow(0 0 4px rgba(0,0,0,0.2));
}
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border-width: 0;
}
#addTab:focus {
    outline: 2px solid #0066cc;
    outline-offset: 2px;
    position: relative;
    z-index: 2;
}
</style>