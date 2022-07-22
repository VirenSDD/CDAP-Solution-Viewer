<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <input type="file" id="selectFiles" accept=".json" /><br />
    <button v-on:click="processSolutionData">Generate visualization</button><br />
    <svg id="grid"></svg>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'InputField',
  props: {
    msg: String
  },
  mounted() {
    d3.select('#grid')
      .attr("style", "outline: thin solid black;")
      .attr('width', window.innerWidth * 0.9)
      .attr('height', window.innerHeight * 0.7);
  },
  data() {
    return {
      solutionData: []
    };
  },
  methods: {
    processSolutionData() {
      d3.select('#grid').selectAll("*").remove();
      const files = document.getElementById("selectFiles").files;

      const fr = new FileReader();

      fr.onload = e => {
        const result = JSON.parse(e.target.result);
        this.solutionData = result;
        console.log(this.solutionData)

        let svg = d3.select('#grid');

        const in_doors = this.solutionData["in_doors"];
        const out_doors = this.solutionData["out_doors"];
        const boundingClientRect = svg.node().getBoundingClientRect();
        const totalHeight = boundingClientRect.height;
        const totalWidth = boundingClientRect.width;

        const gridAdditionInDoor = totalHeight / (in_doors + 1);
        const gridAdditionOutDoor = totalHeight / (in_doors + 1);
        const doorWidth = totalWidth / 5;
        const doorStrokeWidth = 20;

        // INBOUND DOORS
        for (let i = 0; i < in_doors; i++) {
          svg.append('line')
            .attr('stroke', '#76BF8A')
            .attr('stroke-width', doorStrokeWidth)
            .attr("x1", 0)
            .attr("y1", (i + 1) * gridAdditionInDoor)
            .attr("x2", doorWidth)
            .attr("y2", (i + 1) * gridAdditionInDoor);
        }

        // OUTBOUND DOORS
        for (let j = 0; j < out_doors; j++) {
          svg.append('line')
             .attr('stroke', '#76BF8A')
             .attr('stroke-width', doorStrokeWidth)
             .attr("x1", totalWidth)
             .attr("y1", (j + 1) * gridAdditionOutDoor)
             .attr("x2", totalWidth - doorWidth)
             .attr("y2", (j + 1) * gridAdditionOutDoor);
        }
        
        const customers = this.solutionData["customers"];
        const suppliers = this.solutionData["suppliers"];
        let numberSeparation = 20;
        if (suppliers < 10 && customers < 10) {
          numberSeparation /= 2;
        }
        // SUPPLIERS_ASSIGNED
        let suppliersInDoor = new Array(in_doors).fill(0);
        for (let i = 0; i < suppliers; i++) {
          const correspondingDoor = parseInt(this.solutionData["suppliers_assigned"][i]);
          const doorPos = (correspondingDoor + 1) * gridAdditionInDoor + 8;
          const supplierPos = suppliersInDoor[correspondingDoor]++ * (doorWidth / suppliers + numberSeparation);
          svg.append('text')
             .attr('x', supplierPos)
             .attr('y', doorPos)
             .attr('stroke', 'black')
             .style("font-size", doorStrokeWidth)
             .text(i)
        }

        // CUSTOMERS_ASSIGNED
        let customersInDoor = new Array(out_doors).fill(0);
        for (let i = 0; i < customers; i++) {
          const correspondingDoor = parseInt(this.solutionData["customers_assigned"][i]);
          const doorPos = (correspondingDoor + 1) * gridAdditionOutDoor + 8;
          const customerPos = totalWidth - (doorWidth / customers + numberSeparation) * (customersInDoor[correspondingDoor]++ + 1);
          svg.append('text')
             .attr('x', customerPos)
             .attr('y', doorPos)
             .attr('stroke', 'black')
             .style("font-size", doorStrokeWidth)
             .text(i)
        }

        // DELIVERIES
        for (let i = 0; i < suppliers; i++) {
          for (let j = 0; j < customers; j++) {
            if (this.solutionData["deliveries"][i][j]) {
              const supplierPos = (parseInt(this.solutionData["suppliers_assigned"][i]) + 1) * gridAdditionInDoor;
              const customerPos = (parseInt(this.solutionData["customers_assigned"][j]) + 1) * gridAdditionOutDoor;

              svg.append('line')
                 .attr('stroke', 'black')
                 .attr('stroke-width', 1)
                 .attr("x1", doorWidth)
                 .attr("y1", supplierPos)
                 .attr("x2", totalWidth - doorWidth)
                 .attr("y2", customerPos);
            }
          }
        }

        const valueText = "Objective Function Value: " + this.solutionData["objective_function_value"]
        const textSize = 20
        const xText = 0.5 * totalWidth - textSize * valueText.length * 0.25;
        // OBJECTIVE FUNCTION VALUE
        svg.append('text')
           .attr('x', xText)
           .attr('y', totalHeight * 0.99)
           .attr('stroke', 'black')
           .style("font-size", textSize)
           .text(valueText)
      }

      fr.readAsText(files.item(0));
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
