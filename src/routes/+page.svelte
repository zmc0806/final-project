<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let nodes = [{id: 1}, {id: 2}, {id: 3}];
  let links = [
    {source: 0, target: 1},
    {source: 0, target: 2},
    {source: 1, target: 2}
  ];
  let lastNodeId = nodes.length;
  let svg;

  onMount(() => {
    const width = 616, height = 400;
    svg = d3.select("#graph").append("svg")
      .attr("width", width)
      .attr("height", height)
      .on("contextmenu", event => event.preventDefault());

    const force = d3.forceSimulation(nodes)
      .force("link", d3.forceLink(links).id(d => d.id))
      .force("charge", d3.forceManyBody())
      .force("center", d3.forceCenter(width / 2, height / 2));

    force.on("tick", () => {
      vertex.attr("cx", d => d.x)
            .attr("cy", d => d.y);

      edge.attr("x1", d => d.source.x)
          .attr("y1", d => d.source.y)
          .attr("x2", d => d.target.x)
          .attr("y2", d => d.target.y);
    });

    const edge = svg.selectAll(".edge")
      .data(links)
      .enter().append("line")
      .attr("class", "edge");

    const vertex = svg.selectAll(".vertex")
      .data(nodes)
      .enter().append("circle")
      .attr("class", "vertex")
      .attr("r", 10)
      .call(d3.drag()
        .on("start", dragstarted)
        .on("drag", dragged)
        .on("end", dragended));
    
    function dragstarted(event, d) {
      if (!event.active) force.alphaTarget(0.3).restart();
      d.fx = d.x;
      d.fy = d.y;
    }

    function dragged(event, d) {
      d.fx = event.x;
      d.fy = event.y;
    }

    function dragended(event, d) {
      if (!event.active) force.alphaTarget(0);
      d.fx = null;
      d.fy = null;
    }
  });
</script>

<div id="graph"></div>

<style>
  svg {
    cursor: crosshair;
    display: block;
    margin: auto;
  }

  .edge {
    stroke: #888;
    stroke-width: 2px;
    stroke-linecap: round;
    stroke-linejoin: round;
    cursor: default;
  }

  .edge:hover, .dragLine {
    stroke: #333;
    stroke-width: 3px;
  }

  .vertex {
    cursor: pointer;
    fill: steelblue;
  }

  .vertex:hover {
    stroke: #333;
    opacity: 0.8;
  }

  .dragLine.hidden {
    stroke-width: 0;
  }
</style>
