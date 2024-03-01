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

        // 添加显示国家名称的文本元素
        const countryName = svg.append("text")
            .attr("class", "countryName")
            .attr("x", 10) // 设置国家名称文本的初始位置
            .attr("y", 30)
            .style("font-size", "20px")
            .style("fill", "black")
            .style("opacity", 0); // 初始时不显示

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
                    // 添加鼠标事件监听器
                    .on("mouseover", function(d) {
                        d3.select(this).style("fill", "#FFD700"); // Highlight color
                        countryName.style("opacity", 1).text(d.properties.name); // 显示国家名
                    })
                    .on("mouseout", function(d) {
                        d3.select(this).style("fill", "#32CD32"); // Reset to original color
                        countryName.style("opacity", 0); // 隐藏国家名
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

        // 保留您之前的enableRotation、enableDrag和setupZoom函数不变
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

