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
