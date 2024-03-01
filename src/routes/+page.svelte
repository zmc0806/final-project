<!DOCTYPE html>
<html>
<head>
    <style>
        .tooltip {
            position: absolute;
            text-align: center;
            width: auto;
            min-width: 80px; /* Minimum width but can expand */
            height: auto;
            padding: 2px;
            font: 12px sans-serif;
            background: lightsteelblue;
            border: 0px;
            border-radius: 8px;
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            white-space: nowrap; /* Ensure the name is on one line */
        }
    </style>
</head>
<body>
    <svg></svg>
    <div class="tooltip"></div>
    <script src="https://d3js.org/d3.v4.min.js"></script>
    <script src="https://d3js.org/topojson.v1.min.js"></script>
    <script>
        const width = 960;
        const height = 500;
        let isDragging = false;
        const svg = d3.select('svg')
            .attr('width', width).attr('height', height);
        const projection = d3.geoOrthographic()
            .scale(250)
            .translate([width / 2, height / 2]);
        const path = d3.geoPath().projection(projection);
        const graticule = d3.geoGraticule();

        const tooltip = d3.select(".tooltip");

        drawOcean();
        drawGlobe();
        drawGraticule();
        enableRotation();
        enableDrag();
        setupZoom();

        function drawOcean() {
            svg.append("circle")
                .attr("cx", width / 2)
                .attr("cy", height / 2)
                .attr("r", projection.scale())
                .style("fill", "#1E90FF"); // Darker blue for the ocean
        }

        function drawGlobe() {
            d3.json('https://d3js.org/world-110m.v1.json', function(error, world) {
                if (error) throw error;

                svg.selectAll(".segment")
                    .data(topojson.feature(world, world.objects.countries).features)
                    .enter().append("path")
                    .attr("class", "segment")
                    .attr("d", path)
                    .style("stroke", "#FFF") // White borders for contrast
                    .style("stroke-width", "0.5px")
                    .style("fill", "#32CD32") // Greenish land color
                    .on("mouseover", function(d) {
                        tooltip.transition().duration(200).style("opacity", .9);
                        tooltip.html(d.properties.name)
                            .style("left", (d3.event.pageX + 5) + "px")
                            .style("top", (d3.event.pageY - 28) + "px");
                    })
                    .on("mouseout", function(d) {
                        tooltip.transition().duration(500).style("opacity", 0);
                    });
            });
        }

        function drawGraticule() {
            svg.append("path")
                .datum(graticule())
                .attr("class", "graticule")
                .attr("d", path)
                .style("fill", "none")
                .style("stroke", "#DDD") // Light gray graticule for a subtle effect
                .style("stroke-opacity", 0.5);
        }

        function enableRotation() {
            d3.timer(function(elapsed) {
                if (!isDragging) {
                    const rotate = projection.rotate();
                    const k = 1 / 55000;
                    projection.rotate([rotate[0] + k * elapsed, rotate[1]]);
                    svg.selectAll("path").attr("d", path);
                }
            });
        }

        function enableDrag() {
            const drag = d3.drag()
                .on('start', function() {
                    isDragging = true;
                })
                .on('drag', function() {
                    const dx = d3.event.dx;
                    const dy = d3.event.dy;
                    const sensitivity = 0.25; // Adjust for slower drag
                    const rotate = projection.rotate();
                    const scaledDx = dx * sensitivity;
                    const scaledDy = dy * sensitivity;
                    projection.rotate([rotate[0] + scaledDx, rotate[1] - scaledDy]);
                    svg.selectAll("path").attr("d", path);
                })
                .on('end', function() {
                    isDragging = false;
                });

            svg.call(drag);
        }

        function setupZoom() {
            const zoom = d3.zoom()
                .scaleExtent([1, 8])
                .on('zoom', function() {
                    const {transform} = d3.event;
                    const zoomScale = transform.k * 250; // Adjust '250' based on your initial scale
                    projection.scale(zoomScale);
                    svg.selectAll("path.segment").attr("d", path);
                    svg.selectAll("circle").attr("r", projection.scale());
                });

            svg.call(zoom);
        }
    </script>
</body>
</html>
