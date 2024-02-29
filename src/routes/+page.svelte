<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let svg;

  onMount(() => {
    const width = 800, height = 600;
    const graph = {
      nodes: [
        { id: "A" }, { id: "B" }, { id: "C" },
        { id: "D" }, { id: "E" }, { id: "F" }
      ],
      links: [
        { source: "A", target: "B" }, { source: "A", target: "C" },
        { source: "B", target: "D" }, { source: "C", target: "E" },
        { source: "D", target: "E" }, { source: "D", target: "F" },
        { source: "E", target: "F" }
      ]
    };

    const color = d3.scaleOrdinal(d3.schemeCategory10);

    const simulation = d3.forceSimulation(graph.nodes)
      .force('link', d3.forceLink(graph.links).id(d => d.id))
      .force('charge', d3.forceManyBody())
      .force('center', d3.forceCenter(width / 2, height / 2));

    const link = d3.select(svg).append('g')
      .attr('stroke', '#999')
      .selectAll('line')
      .data(graph.links)
      .enter().append('line');

    const node = d3.select(svg).append('g')
      .attr('stroke', '#fff')
      .attr('stroke-width', 1.5)
      .selectAll('circle')
      .data(graph.nodes)
      .enter().append('circle')
        .attr('r', 5)
        .attr('fill', d => color(d.id));

    simulation.on('tick', () => {
      link.attr('x1', d => d.source.x)
          .attr('y1', d => d.source.y)
          .attr('x2', d => d.target.x)
          .attr('y2', d => d.target.y);

      node.attr('cx', d => d.x)
          .attr('cy', d => d.y);
    });
  });
</script>

<svg bind:this={svg} width="800" height="600"></svg>
