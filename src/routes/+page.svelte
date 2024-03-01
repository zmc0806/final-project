<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  onMount(() => {
    const svg = d3.select("#graph")
                  .attr("width", 400)
                  .attr("height", 300);

    const nodes = [{id: 1, x: 100, y: 150}, {id: 2, x: 300, y: 150}];
    const links = [{source: 1, target: 2}];

    const link = svg.selectAll(".link")
                    .data(links)
                    .enter().append("line")
                    .attr("class", "link")
                    .attr("x1", d => nodes[d.source - 1].x)
                    .attr("y1", d => nodes[d.source - 1].y)
                    .attr("x2", d => nodes[d.target - 1].x)
                    .attr("y2", d => nodes[d.target - 1].y)
                    .style("stroke", "#999");

    const node = svg.selectAll(".node")
                    .data(nodes)
                    .enter().append("circle")
                    .attr("class", "node")
                    .attr("cx", d => d.x)
                    .attr("cy", d => d.y)
                    .attr("r", 20)
                    .style("fill", "#69b3a2");
  });
</script>

<svg id="graph"></svg>

<style>
  .link {
    stroke-width: 2px;
  }
  .node {
    stroke: #fff;
    stroke-width: 1.5px;
  }
</style>
