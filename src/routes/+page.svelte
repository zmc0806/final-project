<script>
  // Import necessary libraries and functions
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let svg;
  let traversalType = 'BFS'; // Can be 'BFS' or 'DFS'
  let visited = new Set(); // Tracks visited nodes

  // Sample graph data
  const data = { /* ... */ };

  // Function to initialize the graph
  function initializeGraph() {
    // D3.js graph setup (as before)
  }

  // BFS algorithm
  function bfs(start) {
    let queue = [start];
    while (queue.length > 0) {
      let node = queue.shift();
      if (!visited.has(node)) {
        visited.add(node);
        // Update visualization
        highlightNode(node);
        // Add neighbors to queue
        let neighbors = getNeighbors(node);
        queue.push(...neighbors);
      }
    }
  }

  // DFS algorithm
  function dfs(node) {
    if (!visited.has(node)) {
      visited.add(node);
      // Update visualization
      highlightNode(node);
      // Recurse on neighbors
      let neighbors = getNeighbors(node);
      neighbors.forEach(dfs);
    }
  }

  // Function to highlight a node in the visualization
  function highlightNode(node) {
    d3.select(`#node-${node}`).attr('fill', 'red'); // Example: change fill color
  }

  // Function to get neighbors of a node (implementation depends on data structure)
  function getNeighbors(node) {
    // Return an array of neighbor IDs
    return [];
  }

  onMount(() => {
    initializeGraph();
    // Optionally start BFS or DFS based on user input or automatically
    if (traversalType === 'BFS') bfs('startNode');
    else dfs('startNode');
  });
</script>

<svg bind:this={svg}></svg>

</script>

<svg bind:this={svg}></svg>

