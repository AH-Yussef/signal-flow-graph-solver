<template>
  <div id="solution">
    <div id="solve-btn" @click="triggerSolve()">Solve</div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import { solveforwardpath } from "../../solvingAlgo/forward_paths.js";
import { findLoops } from "../../solvingAlgo/loops.js";
import { transferFunction } from "../../solvingAlgo/Transfer_ function.js";

export default {
  name: 'solve',
  data() {
    return {
      nodeNum: 0,
    }
  },
  computed: mapGetters(['nodes', 'getErrorCard']),
  methods: {
    ...mapActions(['setErrorMsg']),
    triggerSolve() {
      if(!this.isValidGraph()) return;
      const graphInfo = this.getGraphInfo();
      // const graphInfo =  [[0,1,0,0,1,0,0,1,0],
      //                     [0,0,1,0,0,0,1,0,1],
      //                     [1,1,0,1,0,1,0,0,0],
      //                     [0,0,0,0,1,0,0,0,0],
      //                     [0,1,0,0,0,0,0,0,0],
      //                     [0,0,0,1,0,0,0,0,0],
      //                     [0,0,0,0,0,0,0,0,0],
      //                     [0,0,0,0,0,0,0,0,1],
      //                     [0,0,0,0,0,0,0,1,0]];

      this.nodeNum = graphInfo.length;

      const forwardPaths = solveforwardpath(graphInfo);
      const loops = findLoops(graphInfo);

      // const testloops = {
      //   loops : [[3,4,3], [4,5,4], [6,6], [2,6,5,2], [2,3,4,5,2]],
      //   gains : [-10,-4,-1,-20,-100]
      // };
      // const testpaths = {
      //   paths : [[1,2,3,4,5], [1,2,6,5]],
      //   gains : [100, 20]
      // }

      const computations = transferFunction(forwardPaths, loops);

      this.openSolutionArea();
      this.showSolution(forwardPaths, loops, computations);
      document.getElementById("board").style.pointerEvents = 'none';
    },
    getGraphInfo() {
      const matrixSize = this.nodes.size;
      const graphInfo = new Array(matrixSize);

      for(let node of this.nodes.values()) {
        node.unselectSelf();

        const outcomingBranches = node.toConnectionPoint.outConnectors.values();
        const node_nodeGains = new Array(matrixSize).fill(0);
        let toNodeIndex = 0;
        
        for(let branch of outcomingBranches) {
          if(branch.to.isOutputNode) toNodeIndex = matrixSize -1;
          else toNodeIndex = branch.to.code;
          node_nodeGains[toNodeIndex] = branch.gain;
        }

        let nodeIndex = 0;
        if(node.isInputNode) nodeIndex = 0;
        else if(node.isOutputNode) nodeIndex = matrixSize -1;
        else nodeIndex = node.code;

        graphInfo[nodeIndex] = node_nodeGains;
      }

      return graphInfo;
    },
    showSolution(forwardPaths, graphLoops, computations){
      document.getElementById("steps").innerHTML = "";
      this.addTitle("Forward paths :");

      let pathIndex = 1;
      for(let path of forwardPaths.paths) {
        this.addPath(path, pathIndex++, forwardPaths.gains[pathIndex -2]);
      }

      this.addTitle("Loops :");

      let loopIndex = 1;
      for(let loop of graphLoops.loops) {
        this.addLoop(loop, loopIndex++, graphLoops.gains[loopIndex -2]);
      }

      this.addTitle("Transfer function :");

      let deltaIndex = 1;
      for(let delta of computations.deltas) {
        this.addDelta(delta, deltaIndex++);
      }
      this.addDelta(computations.systemDelta, deltaIndex, false);

      this.addFormula(computations.transferFunction);
      this.RenderLatex();

      this.unhighlightAll();
    },
    addTitle(titleText){
      const title = document.createElement('div');
      title.innerHTML = `$$\\underline{\\text{${titleText}}}$$`;
      title.className = "solve-title";
      document.getElementById("steps").appendChild(title);
    },
    addPath(pathSeq, pathIndex, pathGain) {
      const n = pathSeq.length;
      let pathContainer = document.createElement('div');
      let path = document.createElement('div');
      path.innerHTML = `$$P_${pathIndex} :`;

      for(let i = 0; i < n -1; i++) {
        path.innerHTML += `${this._getNodeIndex(pathSeq[i])} \\xrightarrow{}`;
      }
      path.innerHTML += `${this._getNodeIndex(pathSeq[n-1])}$$`;
      path.className = "path";

      let gain = document.createElement('div');
      gain.innerHTML = `$$P_${pathIndex} = ${pathGain}$$`;
      gain.className = "gain";

      pathContainer.appendChild(path);
      pathContainer.appendChild(gain);

      pathContainer.className = "solution-info";

      pathContainer.onmouseover = () => this.highlightSequence(pathSeq);
      pathContainer.onmouseleave = () => this.unhighlightAll();

      document.getElementById("steps").appendChild(pathContainer);
    },
    addLoop(loopSeq, loopIndex, loopGain) {
      const n = loopSeq.length;
      let loopContainer = document.createElement('div');
      let loop = document.createElement('div');
      loop.innerHTML = `$$L_${loopIndex}: `;

      for(let i = 0; i < n -1; i++) {
        loop.innerHTML += `${this._getNodeIndex(loopSeq[i])} \\xrightarrow{}`;
      }
      loop.innerHTML += `${this._getNodeIndex(loopSeq[n-1])}$$`;
      loop.className = "loop";

      let gain = document.createElement('div');
      gain.innerHTML = `$$L_${loopIndex} = ${loopGain}$$`;
      gain.className = "gain";

      loopContainer.appendChild(loop);
      loopContainer.appendChild(gain);

      loopContainer.className = "solution-info";

      loopContainer.onmouseover = () => this.highlightSequence(loopSeq);
      loopContainer.onmouseleave = () => this.unhighlightAll();

      document.getElementById("steps").appendChild(loopContainer);
    },
    addDelta(deltaValue, deltaIndex, addSubIndex = true) {
      let delta = document.createElement('div');
      if(addSubIndex) delta.innerHTML = `$$\\Delta_${deltaIndex} = ${deltaValue}$$`;
      else delta.innerHTML = `$$\\Delta = ${deltaValue}$$`;

      delta.className = "solution-info";
      document.getElementById("steps").appendChild(delta);
    },
    addFormula(transferFunctionValue) {
      let formula = document.createElement('div');
      formula.innerHTML = `$$C(S) / R(S) = ${transferFunctionValue}$$`;
      formula.className = "solution-info";
      document.getElementById("steps").appendChild(formula);
    },
    _getNodeIndex(nodeCode) {
      let nodeIndex = "";
      if(nodeCode == 0) nodeIndex = "R";
      else if(nodeCode == this.nodeNum -1) nodeIndex = "C";
      else nodeIndex = nodeCode;

      return nodeIndex;
    },
    openSolutionArea() {
      this.$root.$refs.solutionArea.open();
      document.getElementById("board").style.transform = "scale(0.5)";
      document.getElementById("board").style.left = "-40%";
    },
    RenderLatex() {
      if(window.MathJax) {
        window.MathJax.Hub.Queue(["Typeset", window.MathJax.Hub]);
      }
    },
    isValidGraph() {
      // any node must have exactly at least one output and at least one input
      for(let node of this.nodes.values()) {
        if(!node.isInputNode && node.fromConnectionPoint.inConnectors.size == 0) {
          this.setErrorMsg(`Node ${node.code} has no input branches`);
          this.getErrorCard.showErrorMsg();
          return false;
        }

        if(!node.isOutputNode && node.toConnectionPoint.outConnectors.size == 0) {
          this.setErrorMsg(`Node ${node.code} has no output brances`);
          this.getErrorCard.showErrorMsg();
          return false;
        }
      }

      return true;
    },
    highlightSequence(sequence) {
      let seq = [...sequence];
      for(let nodeCode of sequence) {
        for(let node of this.nodes.values()) {
          if(this._getNodeCode(node) == nodeCode) {
            node.highlightSelf();
            node.fromConnectionPoint.highlightSelf();
            node.toConnectionPoint.highlightSelf();

            seq.shift();

            for(let branch of node.toConnectionPoint.outConnectors.values()) {
              if(seq.includes(this._getNodeCode(branch.to))) branch.highlightSelf();
            }
          }
        }
      }
    },
    _getNodeCode(node) {
      let nodeCode = 0;
      if(node.isInputNode) nodeCode = 0;
      else if(node.isOutputNode) nodeCode = this.nodeNum -1;
      else nodeCode = node.code;

      return nodeCode;
    },
    unhighlightAll() {
      for(let node of this.nodes.values()) {
        node.unselectSelf();
        node.fromConnectionPoint.unselectSelf();
        node.toConnectionPoint.unselectSelf();

        for(let branch of node.toConnectionPoint.outConnectors.values()) {
          branch.unselectSelf();
        }
      }
    }
  },
}
</script>

<style scoped>
#solution {
  height: 100%;
  width: 10rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

#solve-btn { 
  background-color: rgb(30, 89, 202);
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