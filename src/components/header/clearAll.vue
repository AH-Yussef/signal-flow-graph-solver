<template>
  <div id="clear-all">
    <div id="delete-all-btn" @click="clearAll()">clear all</div>
  </div>
</template>

<script>
import { Node } from '../../graphComponents/node.js';
import { mapGetters, mapActions } from 'vuex';

export default {
  name: 'clearAll',
  data() {
    return {

    }
  },
  computed: mapGetters(['nodes']),
  methods: {
    ...mapActions(['resetCurrSelectedComponent']),
    clearAll() {
      for(let node of this.nodes.values()){
        node.remove();
      }
      Node.codeCount = 1;
      Node.availableCodes.length = 0;

      this.resetCurrSelectedComponent();
    }
  }
}
</script>

<style>
#clear-all {
  height: 100%;
  width: 10rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

#delete-all-btn {
  background-color: #E74C3C;
  border-radius: 0.5rem;
  height: 2rem;
  width: 6rem;

  text-align: center;
  vertical-align: center;
  line-height: 2rem;

  color: white;
  user-select: none;
}
</style>