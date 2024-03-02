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
        let countryCasesMap = {}; // 存储国家 ISO 代码和 COVID-19 总病例数的映射

        // 加载 CSV 数据
        d3.csv("gdp-per-capita-worldbank.csv", function(error, csvData) {
            if (error) throw error;

            // 填充 countryCasesMap
            csvData.forEach(function(d) {
                countryCasesMap[d.iso_code] = d.total_cases;
            });

            // 一旦数据加载完毕，绘制地图和相关功能
            drawOcean();
            drawGlobe();
            drawGraticule();
            enableRotation();
            enableDrag();
            setupZoom();
        });

        function drawOcean() {
            svg.append("circle")
                .attr("cx", width / 2)
                .attr("cy", height / 2)
                .attr("r", projection.scale())
                .style("fill", "#1E90FF"); // Darker blue for the ocean
        }

        function drawGlobe() {
            d3.json('world-110m_with_names.v1.json', function(error, world) {
                if (error) throw error;

                // 绘制国家轮廓
                const countries = topojson.feature(world, world.objects.countries).features;
                svg.selectAll(".segment")
                    .data(countries)
                    .enter().append("path")
                    .attr("class", "segment")
                    .attr("d", path)
                    .style("stroke", "#FFF") // 白色边界以增加对比度
                    .style("stroke-width", "0.5px")
                    .style("fill", "#32CD32") // 浅绿色陆地
                    .on("mouseover", function(d) {
                        d3.select(this).style("fill", "#FFD700"); // 高亮颜色
                        showCountryName(d.properties.name, d.id); // 显示国家名称和 ISO 代码
                    })
                    .on("mouseout", function(d) {
                        d3.select(this).style("fill", "#32CD32"); // 重置为原始颜色
                        hideCountryName();
                    });


            });
        }



        function showCountryName(name, isoCode) {
            const cases = countryCasesMap[isoCode] || 'No data';
            svg.append("text")
                .attr("class", "countryName")
                .attr("x", 20)
                .attr("y", 35)
                .text(`${name}: ${cases} cases`)
                .style("font-size", "20px")
                .style("fill", "black");
        }

        function hideCountryName() {
            svg.selectAll(".countryName").remove();
        }

        function drawGraticule() {
            svg.append("path")
                .datum(graticule())
                .attr("class", "graticule")
                .attr("d", path)
                .style("fill", "none")
                .style("stroke", "#DDD")
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
                    const rotate = projection.rotate();
                    projection.rotate([rotate[0] + dx, rotate[1] - dy]);
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
                    svg.attr("transform", d3.event.transform);
                });

            svg.call(zoom);
        }
    </script>
</body>
</html>
