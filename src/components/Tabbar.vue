<script>
import Tab from './Tab.vue';
let uniqueId = 0;
export default {
    name: 'Tabbar',
    components: { Tab },
    data() {
        return {
            inputValue: '',
            tabs: [],
            draggedTabId: null,
            isFirstDragged: false,
            activeTab: null,
            liveAnnouncement: '',
            lastKeyboardTabFocus: null,
            curretHoveredElement: null,
            isDialogue: false,
            selectedProperty: null,
            incremented: false,
            isOverFlowDialogOpen: false,
            isChartSelect: false,
            isTableSelect:false,
            overflowDialogList: [],
            dialogHideFlag: false,
            lastFocusedTabId: null,
            trueLastFocusedTabId:null
        };
    },
    methods: {
        addTabAction() {
            this.isDialogue = true;
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
            window.mounted
        },
        closeAllTabAction() {

            this.tabs = [];
            this.overflowDialogList = [];
            this.liveAnnouncement = 'Closed all tabs';
            this.activeTab = null;
            this.incremented = false;
            document.getElementById("tabOverflowDialog").close();
        },
        closeTabAction(id) {
            const tab = this.tabs.find(t => t.uid === id);
            const overflowTab = this.overflowDialogList.find(o => o.uid === id);
            this.tabs = this.tabs.filter(tab => tab.uid !== id);
            this.overflowDialogList = this.overflowDialogList.filter(overflowTab => overflowTab.uid !== id);
            
            this.liveAnnouncement = `Closed tab ${tab.content}`;
            if (this.activeTab === id) {
                this.activeTab = this.tabs.length > 0 ? this.tabs[this.tabs.length - 1].uid : null;
            }
            if (this.tabs.length < 12) {
                this.incremented = false;
            }
        },
        dragTabAction(event, id) {
    this.activeTab = id;
    const tabElements = document.querySelectorAll('.tab-wrapper');
    tabElements.forEach(el => el.classList.add('hovered'));
    this.draggedTabId = id;
    event.dataTransfer.effectAllowed = 'move';
    event.dataTransfer.setData('text/plain', id.toString());

    const dragElement = event.target.closest('.tab-wrapper');
    dragElement.classList.add('dragging');

    const ghost = dragElement.cloneNode(true);
    ghost.id = 'drag-ghost';
    const tab_wrapper = ghost.querySelector('.tab-wrapper'); // Adjust selector if needed


    ghost.style.position = 'fixed';
    ghost.style.backgroundColor = '#ffffff';
    ghost.style.pointerEvents = 'none';
    ghost.style.zIndex = '9999';
    ghost.style.opacity = '1';
    ghost.style.transition = 'transform 0.1s ease-out';
    ghost.style.width = `${dragElement.offsetWidth}px`;

    const rect = dragElement.getBoundingClientRect();
    ghost.style.left = `${rect.left + rect.width / 2}px`;
    ghost.style.top = `${rect.top + rect.height / 2}px`;
    ghost.style.transform = 'translate(-50%, -50%) perspective(800px)';

    document.body.appendChild(ghost);

    let prevX = event.clientX;
    let prevY = event.clientY;

    const moveGhost = (e) => {
        const deltaX = e.clientX - prevX;
        const deltaY = e.clientY - prevY;
        prevX = e.clientX;
        prevY = e.clientY;

        const rotateX = -deltaY * 0.4;
        const rotateY = deltaX * 0.3;

        requestAnimationFrame(() => {
            ghost.style.left = `${e.clientX}px`;
            ghost.style.top = `${e.clientY}px`;
            ghost.style.transform = `
                translate(-50%, -50%)
                perspective(800px)
                rotateX(${rotateX*5}deg)
                rotateY(${rotateY*5}deg)
            `;
        });
    };

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
            document.querySelectorAll('.tab-wrapper.hovered').forEach(tab => {
                if (tab !== this.currentHoveringElement) {
                    tab.classList.remove('hovered');
                }
            });
            const firstHoveredTab = document.querySelector('.tab-wrapper.hovered');
            if (firstHoveredTab && firstHoveredTab !== this.currentHoveringElement) {
                firstHoveredTab.classList.remove('hovered');
            }
            event.target.closest('.tab-wrapper').classList.remove('dragging');
            this.draggedTabId = null;
            const tabElement = document.querySelector(`.tab-wrapper[data-id="${this.activeTab}"]`);
            if (tabElement) {
                tabElement.focus();
            }
        },
        handleDragOverTab(event, hoveredTabId) {
            event.preventDefault();
            if (this.draggedTabId === null || this.draggedTabId === hoveredTabId) return;
            const draggedIndex = this.tabs.findIndex(tab => tab.uid === this.draggedTabId);
            const hoveredIndex = this.tabs.findIndex(tab => tab.uid === hoveredTabId);
            if (draggedIndex === hoveredIndex) return;
            const hoveredElement = event.currentTarget;
            hoveredElement.classList.add('hovered');
            this.curretHoveredElement = hoveredElement;
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
            const focusedElement = document.activeElement;
            let currentIndex = -1;
            if (focusedElement.classList.contains('tab-wrapper')) {
                currentIndex = this.tabs.findIndex(tab => tab.uid.toString() === focusedElement.dataset.id);
            }
            else if (focusedElement.id === 'addTab') {
                return;
            }
            if (currentIndex === this.tabs.length - 1 || currentIndex === -1) {
                e.preventDefault();
                document.getElementById('addTab').focus();
            } else {
                e.preventDefault();
                this.focusTab(this.tabs[currentIndex + 1].uid);
            }
        },
        
        handleShiftTabKey(e) {
            if (this.tabs.length === 0) {
                document.getElementById('addTab').focus();
                return;
            }
            const focusedElement = document.activeElement;
            let currentIndex = -1;
            if (focusedElement.classList.contains('tab-wrapper')) {
                const focusedTabId = focusedElement.dataset.id;
                currentIndex = this.tabs.findIndex(tab => tab.uid.toString() === focusedTabId);
            }


            if (focusedElement.id === 'addTab') {
                e.preventDefault();
                this.focusTab(this.tabs[this.tabs.length - 2].uid);
                return;
            }
            if (currentIndex === 0) {
                this.focusTab(currentIndex - 1);
                return;
            }
            if (currentIndex > 0) {
                e.preventDefault();
                this.focusTab(this.tabs[currentIndex - 1].uid);
                return;
            }
            if (currentIndex < -1) {
            }
            document.querySelector(`.tab-wrapper[data-id="${this.tabs.length - 2}"]`).focus();
        },
        handleTabOut(e) {
            if (this.tabs.length > 0) {
                e.preventDefault();
                this.focusTab(this.tabs[0].uid);
            }
        },
        handleShiftTabIn(e) {
            if (this.tabs.length > 0) {
                e.preventDefault();
                this.focusTab(this.tabs[this.tabs.length - 1].uid);
            }
        },
        handleKeydown(e, tabId) {
            const tabCount = this.tabs.length;
            const currentIndex = this.tabs.findIndex(tab => tab.uid === tabId);
            switch (e.key) {
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
        focusTab(tabId, islast = false) {
            if (tabId === 'addTab') {
                this.$nextTick(() => {
                    document.getElementById('addTab').focus();
                    return;
                });
            }
            if (tabId === -1) {
                this.$nextTick(() => {
                    document.getElementById('addTab').focus();
                });
            }
            else {
                this.activeTab = tabId;
                this.$nextTick(() => {
                    const tabEl = document.querySelector(`.tab-wrapper[data-id="${tabId}"]`);
                    if (tabEl) {
                        tabEl.focus();
                        tabEl.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
                    }
                });
            }
        },
        methods: {
            toggleDialog() {
                const dialog = this.$refs.addDialog;
                if (dialog.open) {
                    dialog.close();
                } else {
                    dialog.show();
                    this.positionDialog();
                }
            },
            closeDialog() {
                this.$refs.addDialog.close();
            },
            positionDialog() {
                const button = document.getElementById('addTab');
                const dialog = this.$refs.addDialog;
                if (button && dialog) {
                    const buttonRect = button.getBoundingClientRect();
                    dialog.style.top = `${buttonRect.bottom + window.scrollY}px`;
                    dialog.style.left = `${buttonRect.left + window.scroll}px`;
                }
            },
        },
        mounted() {
            window.addEventListener('resize', this.positionDialog);
        },
        beforeDestroy() {
            window.removeEventListener('resize', this.positionDialog);
        },
        dialogAction() {
            if (!this.isDialogue) {
                document.getElementById('tabInsertPopup').show()
            }
            else {
                this.cancelDialogBox();
            }
            this.isDialogue = !this.isDialogue;
        },
        selectProperty(propertyName,name) {
            this.selectedProperty = propertyName;
            this.isDialogue = false;
            const newTab = {
                uid: uniqueId++,
                content: name || `Untitled ${uniqueId}`,
                property: propertyName  
            };
            this.inputValue = '';
            if (this.tabs.length >= 5) {
                this.overflowDialogList.push(newTab);
                document.getElementById("addTab").style.marginLeft = 0 + 'px';
                this.incremented = true;
            }
            else if (this.tabs.length < 5) {
            }
            
            document.getElementById('tabInsertPopup').close();
            this.isDialogue = false;
            this.tabs.push(newTab);
            this.activeTab = newTab.uid;
            this.$nextTick(() => {
                const tabEl = document.querySelector(`.tab-wrapper[data-id="${newTab.uid}"]`);
                if (tabEl) {
                    tabEl.focus();
                }
            });
        },
        overflowDialogButtonAction(overFlowedTab) {
            console.log(`fired: ${overFlowedTab.content} - ${overFlowedTab.uid} `);
            document.querySelector(`.tab-wrapper[data-id="${overFlowedTab.uid}"]`).focus();
        },
        toggleTabOverflowDialog() {
            if (this.isOverFlowDialogOpen) {
                document.getElementById("tabOverflowDialog").close();
                this.isOverFlowDialogOpen = false;
                document.querySelector(`.tab-wrapper[data-id="${this.lastFocusedTabId}"]`).focus();
            }
            else {
                document.getElementById("tabOverflowDialog").show();
                this.isOverFlowDialogOpen = true;
            }
        },
        chartSelect(){
            if(this.isChartSelect){
                this.isChartSelect = false;
            }
            else{
                this.isChartSelect = true;
                this.isTableSelect = false;
            }


        },
        tableSelect(){
            if(this.isTableSelect){
                this.isTableSelect = false;
            }
            else{
                this.isTableSelect = true;
                this.isChartSelect = false;
            }
        },
          async cancelDialogBox() {

              const dialog = document.getElementById('tabInsertPopup');
              
              dialog.classList.add('hiding');
              
              await new Promise(resolve => {
                  dialog.addEventListener('animationend', () => {
                      dialog.close();
                      dialog.classList.remove('hiding');
                      resolve();
                    }, { once: true }); 
                });
                
                dialog.close();
                this.isDialogue = false;
                this.isChartSelect = false;
                this.isTableSelect = false;
                this.inputValue = '';
                // this.focusTab(document.querySelector(`.tab-wrapper[data-id="${this.lastFocusedTabId}"]`));
                if(this.tabs.length){
                    document.querySelector(`.tab-wrapper[data-id="${this.lastFocusedTabId}"]`).focus();
                }
                else{
                    document.getElementById('addTab').focus();
                }
  }
    },
    mounted() {
        window.Access = this;
        window.getTabInfo = () => Access.tabs;
        window.getTabInfoById = id => Access.tabs.find(tab => tab.uid === id);
        window.lastFocusedTab = document.addEventListener('focusin',e => {
            e.target.dataset.id?this.lastFocusedTabId = e.target.dataset.id : false;
            console.log(this.lastFocusedTabId);
        })
    }
};
</script>

<template>
    <nav aria-label="Tab navigation">
        <div class="tabbar">
            <transition-group name="reorder" tag="div" class="tabs-wrapper" role="tablist"
                @keydown.tab.prevent="handleTabKey" @keydown.shift.tab.prevent="handleShiftTabKey">
                <div v-for="tab in tabs" :key="tab.uid" class="tab-wrapper" draggable="true" :data-id="tab.uid"
                    role="tab" :aria-selected="activeTab === tab.uid" :tabindex="activeTab === tab.uid ? 0 : -1"
                    @dragstart="(e) => dragTabAction(e, tab.uid)" @dragend="dragEndAction"
                    @dragover.prevent="(e) => handleDragOverTab(e, tab.uid)" @keydown="(e) => handleKeydown(e, tab.uid)"
                    @click="focusTab(tab.uid)">
                    <Tab :id="tab.uid" :content="tab.content" :property="tab.property" @close="closeTabAction" />
                </div>
            </transition-group>
            <div class="overflowed-div">
                <button v-if="incremented" id="tabOverflowBtn" @click="toggleTabOverflowDialog"><i class="fa fa-caret-down" aria-hidden="true"></i></button>
                <dialog id="tabOverflowDialog" class="overflow-dialog-div">
                    <div style="display: flex; justify-content: space-between;">
                        <p>Overflowed Tabs</p>
                        <button
                            style="display: flex; width: 5px; height: 5px; justify-content: center; align-items: center; position: absolute; right: 20px; top: 20px; background-color: white; border: 0; cursor: pointer;"
                            @click="toggleTabOverflowDialog">x</button>
                    </div>
                    <button v-for="overFlowedTab in overflowDialogList" :key="overFlowedTab.uid"
                        @click="overflowDialogButtonAction(overFlowedTab)">{{ overFlowedTab.content }}</button>
                    <button @click="closeAllTabAction">close all</button>
                </dialog>
            </div>
            <div style="display: inline; position: relative; position: absolute; right: 10px;">
                <button id="addTab" aria-label="Add new tab" @click="dialogAction" @keydown.tab="handleTabOut" @keydown.shift.tab="handleShiftTabIn" tabindex="0">+</button>
            </div>
        </div>
        <div class="sr-only">{{ liveAnnouncement }}</div>
    </nav>
    <dialog id="tabInsertPopup">
        <div class="w100">
            <h2>Tag Name</h2>
        </div>
        <div class="w100 tm">
            <input type="text" placeholder="Enter Tag Name" v-model="inputValue" style="margin: 0; width: 300px; text-align: center; height: 20px; padding: 10px; border: 0; border-radius: 5px;"/>
            <div>
                <button class="propertyBtn" id="chartSelect" :style="`background-color: ${isChartSelect? 'dodgerblue': '#2f2f2f'}`" @click="chartSelect" >Chart</button>
                <button class="propertyBtn" id="tableSelect" :style="`background-color: ${isTableSelect? 'dodgerblue': '#2f2f2f'}`" @click="tableSelect">Table</button>
            </div>
        </div>
        <div class="w100">
            <button class="Btn" @click="selectProperty(isChartSelect?'chart': isTableSelect?'excel':null, inputValue); isChartSelect=false; isTableSelect=false;">Add</button>
            <button class="Btn" style="background-color: #afafaf;" @click="cancelDialogBox">Cancel</button> 
        </div>
    </dialog>
</template>


<style scoped>

.Btn{
    border: 0;
    padding: 10px;
    border-radius: 5px;
}

.propertyBtn{
    color: aliceblue;
    font-weight:600;
    padding: 10px;
    border: 0;
    border-radius: 5px;
    margin: 5px 10px;
}
.w100{
    margin-top: 5px;
    display: flex;
    padding: 20px;
    gap: 20px;
    width: 100%-100px;
    justify-content: center;
    align-items: center;
}
#tabInsertPopup{
    display: flexbox;
    border: 0;
    backdrop-filter: blur(10px);
    margin: 25vh auto 25vh auto;
    background-color: rgb(216, 216, 216);
    width: 40vw;
    height: 31vh;
    justify-content: center;
    align-items: center;
    opacity: 95%;
}
.tab-wrapper {
    user-select: none;
    outline: none;
    position: relative;
}

#drag-ghost .tab :nth-child(1){
    color: dodgerblue;
}
#drag-ghost {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    filter: drop-shadow(0 0 4px rgba(0, 0, 0, 0.2));
}

.overflowed-div {
    position: absolute;
    align-items: center;
    right: 40%;
    top: 13.5px;
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
    outline: 2px solid #0066cc00;
    outline-offset: 2px;
    position: relative;

    border: 0;
    z-index: 2;
    animation: wobbleScale 4s ease;
    animation-iteration-count: infinite;

}
#tabOverflowDialog {
    position: absolute;
    margin-left: -180px;
    margin-top: 10px;
    width: 350px;
    border: 0;
    padding: 10px 10px;
}
#tabOverflowDialog button {
    width: 100%;
    padding: 5px;
    margin: 5px 0;
    cursor: pointer;
}

dialog{
    animation: addDialogueAnimation 0.2s ease-in;
}
#tabInsertPopup.hiding{
    animation: addDialogueAnimationRev 0.2s ease-in;
}
button{
    offset: 0;
}
button:active{
    offset: 0 20px;
}

@keyframes addDialogueAnimation{
    from {
        transform: scale(0.5);
        opacity: 0;
    }
    to {
        transform: scale(1);
        opacity: 1;
    }
}
@keyframes addDialogueAnimationRev{
    from {
        transform: scale(1);
        opacity: 1;
    }
    to {
        transform: scale(0.5);
        opacity: 0;
    }
}

@keyframes wobbleScale {
    0% {
        transform: scale(1);
        background-color: #a0a0a0;
    }

    15% {
        transform: scale(1.05);
        background-color: #bdbbbb;
    }

    30% {
        transform: scale(0.95);
        background-color: #a0a0a0;
    }

    45% {
        transform: scale(1.05);
        background-color: #bdbbbb;
    }

    60% {
        transform: scale(0.95);
        background-color: #a0a0a0;
    }

    75% {
        transform: scale(1.05);
        background-color: #bdbbbb;
    }

    100% {
        transform: scale(1);
        background-color: #a0a0a0;
    }
}

.add-dialog {
    position: absolute;
    margin: 0;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.dialog-div {
    border: 0;
    padding: 10px 10px;
    margin: 10px 0 0 -180px;
}

.dialog-div button {
    width: 100%;
    padding: 5px;
    margin: 5px 0;
}

.tabs-wrapper {
    overflow-x: auto;
    white-space: nowrap;
}

.tabs-wrapper::-webkit-scrollbar {
    height: 0px;
}

.tabs-wrapper::-webkit-scrollbar-thumb {
    background-color: #ccc;
    border-radius: 10px;
}

.tabs-wrapper::-webkit-scrollbar-track {
    background-color: #fff;
}

#tabOverflowBtn {
    display: inline;
    margin-left: -180px;
}
</style>