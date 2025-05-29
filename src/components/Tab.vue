<script>
export default {
  name: 'Tab',
  props: ['id', 'content','property'],
  data: function(){
    return {
      propertyIcon: ["fa-pie-chart","fa-table","fa-exclamation-circle"]
    }

  },  
  computed: {
  propertyNumber() {
    return this.property === 'chart' ? 0 : this.property === 'excel'? 1 : 2;
  }
},
  methods: {
      getUrl(url){
        return new URL(url, import.meta.url).href
      },
    }
};
</script>

<template>
  <div class="tab">
    <!-- <img src="../assets/shopping-cart.svg" width="15px" height="15px" background-color="inherit"/> -->
    <!-- <img :src="getUrl(propertyIcon[propertyNumber])" width="15px" height="15px" background-color="inherit"/> -->
    <i :class="`fa ${propertyIcon[propertyNumber]}`" width="15px" height="15px" aria-hidden="true"></i>

    <span>{{ content }}</span>

      <button 
      id="closeBtn" 
      aria-label="Close tab"
      tabindex="-1"
      @click.stop="$emit('close', id)"
      @keydown.stop
      @mousedown.stop


    >×</button>


  </div>
</template>

<style scoped>

.tab {
  padding: 10px;
  background-color: var(--tabbar);
  user-select: none;
  color: var(--def-font);
}

#closeBtn {
  background: none;
  font-family: Arial, Helvetica, sans-serif;
  border: none;
  cursor: pointer;
  font-size: 1rem;
  pointer-events: none;
  margin-left: 8px;
  visibility: hidden;
}

.tab:hover #closeBtn,
.tab:focus-within #closeBtn {
  pointer-events: auto;
}

#closeBtn:focus {
  outline: none;
}


/*----------------------------------*/

#closeBtn:active {
  color: red;
}
#closeBtn:focus {
  color: red;
}

/*----------------------------------*/

</style>