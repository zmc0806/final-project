<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let svg;

  onMount(() => {
    const data = {
      nodes: [{ id: "1" }, { id: "2" }, { id: "3" }],
      links: [{ source: "1", target: "2" }, { source: "2", target: "3" }]
    };

    const width = 400, height = 200;
    const simulation = d3.forceSimulation(data.nodes)
                         .force("link", d3.forceLink(data.links).id(d => d.id))
                         .force("charge", d3.forceManyBody())
                         .force("center", d3.forceCenter(width / 2, height / 2));

    const svgElement = d3.select(svg)
                         .attr("width", width)
                         .attr("height", height);

    const link = svgElement.append("g")
                           .attr("stroke", "#999")
                           .selectAll("line")
                           .data(data.links)
                           .join("line")
                           .attr("stroke-opacity", 0.6);

    const node = svgElement.append("g")
                           .attr("stroke", "#fff")
                           .attr("stroke-width", 1.5)
                           .selectAll("circle")
                           .data(data.nodes)
                           .join("circle")
                           .attr("r", 5)
                           .attr("fill", colorNode)
                           .call(drag(simulation));

    node.append("title")
        .text(d => d.id);

    simulation.on("tick", () => {
      link.attr("x1", d => d.source.x)
          .attr("y1", d => d.source.y)
          .attr("x2", d => d.target.x)
          .attr("y2", d => d.target.y);

      node.attr("cx", d => d.x)
          .attr("cy", d => d.y);
    });

    function colorNode(d) {
      return d.id === "1" ? "red" : "blue"; // Example coloring function
    }

    function drag(simulation) {
      function dragstarted(event) {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        event.subject.fx = event.subject.x;
        event.subject.fy = event.subject.y;
      }

      function dragged(event) {
        event.subject.fx = event.x;
        event.subject.fy = event.y;
      }

      function dragended(event) {
        if (!event.active) simulation.alphaTarget(0);
        event.subject.fx = null;
        event.subject.fy = null;
      }

      return d3.drag()
               .on("start", dragstarted)
               .on("drag", dragged)
               .on("end", dragended);
    }
  });
</script>

<svg bind:this={svg}></svg>

