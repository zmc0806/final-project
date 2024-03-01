<!-- SVMVisualization.svelte -->
<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let svg;

  onMount(() => {
    const width = 800, height = 600;
    const svg = d3.select(svg)
      .attr('width', width)
      .attr('height', height);

    // Sample data - replace this with your SVM output
    const dataPoints = [{x: 100, y: 200, class: 0}, {x: 150, y: 300, class: 1}];
    const decisionBoundary = [{x1: 0, y1: 100, x2: 800, y2: 500}];
    
    // Render data points
    svg.selectAll('.data-point')
      .data(dataPoints)
      .enter().append('circle')
        .attr('class', 'data-point')
        .attr('cx', d => d.x)
        .attr('cy', d => d.y)
        .attr('r', 5)
        .style('fill', d => d.class === 0 ? 'blue' : 'red');

    // Render decision boundary
    svg.selectAll('.decision-boundary')
      .data(decisionBoundary)
      .enter().append('line')
        .attr('x1', d => d.x1)
        .attr('y1', d => d.y1)
        .attr('x2', d => d.x2)
        .attr('y2', d => d.y2)
        .style('stroke', 'black')
        .style('stroke-width', 2);
  });
</script>

<svg bind:this={svg}></svg>

