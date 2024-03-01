<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let svgElement;
  let force;
  let dragLine;
  let width = 616, height = 400; // 默认SVG尺寸

  let nodes = [
    { id: 1, x: 150, y: 100 },
    { id: 2, x: 300, y: 100 },
    { id: 3, x: 450, y: 100 },
  ];

  let links = [
    { source: 0, target: 1 },
    { source: 1, target: 2 },
  ];

  let lastNodeId = nodes.length;

  // D3颜色比例尺
  let color = d3.scaleOrdinal(d3.schemeCategory10);

  function restart() {
    const svg = d3.select(svgElement);

    // 清除旧的SVG元素
    svg.selectAll('*').remove();

    // 创建力导向图实例
    force = d3.forceSimulation(nodes)
      .force("link", d3.forceLink(links).id(d => d.id).distance(100))
      .force("charge", d3.forceManyBody().strength(-500))
      .force("center", d3.forceCenter(width / 2, height / 2));

    // 绘制边
    svg.append("g")
      .attr("class", "links")
      .selectAll("line")
      .data(links)
      .join("line")
      .style("stroke", "#aaa")
      .style("stroke-width", 2);

    // 绘制节点
    const node = svg.append("g")
      .attr("class", "nodes")
      .selectAll("circle")
      .data(nodes)
      .join("circle")
      .attr("r", 10)
      .style("fill", d => color(d.id))
      .call(drag(force));

    // 节点和边的坐标更新
    force.on("tick", () => {
      svg.selectAll(".links line")
        .attr("x1", d => d.source.x)
        .attr("y1", d => d.source.y)
        .attr("x2", d => d.target.x)
        .attr("y2", d => d.target.y);

      node.attr("cx", d => d.x)
          .attr("cy", d => d.y);
    });
  }

  // D3拖拽配置
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

  onMount(() => {
    restart();
  });
</script>

<svg bind:this={svgElement} width={width} height={height}></svg>

