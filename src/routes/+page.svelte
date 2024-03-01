<!DOCTYPE html>
<html>
<body>
    <svg></svg>
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

                const countries = topojson.feature(world, world.objects.countries).features;
                svg.selectAll(".segment")
                    .data(countries)
                    .enter().append("path")
                    .attr("class", "segment")
                    .attr("d", path)
                    .style("stroke", "#FFF") // White borders for contrast
                    .style("stroke-width", "0.5px")
                    .style("fill", "#32CD32") // Greenish land color
                    .on("mouseover", function(d) {
                        d3.select(this).style("fill", "#FFD700"); // Highlight color
                        showCountryName(d.properties.name);
                    })
                    .on("mouseout", function(d) {
                        d3.select(this).style("fill", "#32CD32"); // Reset to original color
                        hideCountryName();
                    });
            });
        }

        // 添加新的函数来显示和隐藏国家名
        function showCountryName(name) {
            svg.append("text")
                .attr("class", "countryName")
                .attr("x", 20) // 根据需要调整文本位置
                .attr("y", 35)
                .text(name)
                .style("font-size", "20px")
                .style("fill", "black");
        }

        function hideCountryName() {
            svg.selectAll(".countryName").remove(); // 移除国家名的文本元素
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
                .on('zoom', function(event) {
                    const {transform} = event;
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

