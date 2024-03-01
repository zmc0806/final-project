<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let vertices = [];
  let edges = [];
  let svg;

  onMount(() => {
    const width = 800, height = 600;

    svg = d3.select("#graph").append("svg")
      .attr("width", width)
      .attr("height", height)
      .on("contextmenu", event => {
        event.preventDefault();
      })
      .on("click", addVertex);

    function addVertex(event) {
      if (event.ctrlKey) return;
      const [x, y] = d3.pointer(event);
      const vertex = {x, y, id: Date.now()};
      vertices.push(vertex);
      updateGraph();
    }

    function addEdge(startVertex) {
      return function(event) {
        const endVertex = vertices.find(v => v.id == event.subject.id);
        edges.push({source: startVertex, target: endVertex});
        updateGraph();
      }
    }

    function updateGraph() {
      const vertexSelection = svg.selectAll(".vertex")
        .data(vertices, d => d.id);

      vertexSelection.enter()
        .append("circle")
        .attr("class", "vertex")
        .attr("r", 10)
        .attr("cx", d => d.x)
        .attr("cy", d => d.y)
        .call(d3.drag().on("start", addEdge))
        .merge(vertexSelection)
        .on("contextmenu", event => {
          const id = event.subject.id;
          vertices = vertices.filter(v => v.id !== id);
          edges = edges.filter(e => e.source.id !== id && e.target.id !== id);
          updateGraph();
          event.preventDefault();
        });

      vertexSelection.exit().remove();

      const edgeSelection = svg.selectAll(".edge")
        .data(edges, d => `${d.source.id}-${d.target.id}`);

      edgeSelection.enter()
        .append("line")
        .attr("class", "edge")
        .attr("x1", d => d.source.x)
        .attr("y1", d => d.source.y)
        .attr("x2", d => d.target.x)
        .attr("y2", d => d.target.y)
        .merge(edgeSelection);

      edgeSelection.exit().remove();
    }
  });
</script>

<div id="graph"></div>

<style>
  svg { border: 1px solid black; }
  .vertex { fill: steelblue; cursor: pointer; }
  .edge { stroke: #999; stroke-opacity: 0.6; }
</style>
