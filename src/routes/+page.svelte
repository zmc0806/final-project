<!DOCTYPE html>
<html>
<body>
    <!-- Dropdown for selecting a year -->
    <label for="yearSelector">Select Year:</label>
    <select id="yearSelector"></select>
    <svg></svg>
    <!-- D3.js and TopoJSON for map rendering -->
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
        let countryCasesMap = {};
        let csvData; // Global variable to store CSV data
        const barChartWidth = 500;
        const barChartHeight = 300;
        const margin = { top: 20, right: 20, bottom: 30, left: 40 };

        const barSvg = d3.select('body').append('svg')
            .attr('width', barChartWidth)
            .attr('height', barChartHeight);



        // Load CSV data
        d3.csv("gdp-per-capita-worldbank.csv", function(error, data) {
            if (error) throw error;
            csvData = data; // Store the CSV data globally

            // Populate the year selector with unique years from the data
            const years = Array.from(new Set(data.map(d => d.Year))).sort();
            const yearSelector = d3.select("#yearSelector");
            years.forEach(year => {
                yearSelector.append("option").text(year).attr("value", year);
            });

            // Initialize the map with the first available year
            updateMapForYear(years[0]);
        });

        // Update map based on selected year
        // Update map based on selected year
        function updateMapForYear(selectedYear) {
            countryCasesMap = {}; // Reset the map
            csvData.filter(d => d.Year == selectedYear).forEach(d => {
                countryCasesMap[d.Entity] = +d["GDP per capita, PPP (constant 2017 international $)"];
            });

            // Clear previous globe drawings and redraw
            svg.selectAll(".segment").remove(); // Clear previous countries
            svg.selectAll(".graticule").remove(); // Clear previous graticules
            svg.selectAll(".countryName").remove(); // Clear country names

            // Redraw map elements
            drawOcean();
            drawGlobe();
            drawGraticule();
            // Re-enable rotation after updating
            enableRotation();
            enableDrag();
        }


        // Listen for year selection changes
        d3.select("#yearSelector").on("change", function() {
            updateMapForYear(this.value);
        });

        // Function to draw the ocean
        function drawOcean() {
            svg.append("circle")
                .attr("cx", width / 2)
                .attr("cy", height / 2)
                .attr("r", projection.scale())
                .style("fill", "#1E90FF"); // Ocean color
        }

        // Function to draw the globe
        function drawGlobe() {
            d3.json('world-110m_with_names.v1.json', function(error, world) {
                if (error) throw error;

                const countries = topojson.feature(world, world.objects.countries).features;
                svg.selectAll(".segment")
                    .data(countries)
                    .enter().append("path")
                    .attr("class", "segment")
                    .attr("d", path)
                    .style("stroke", "#FFF") // Country borders
                    .style("stroke-width", "0.5px")
                    .style("fill", d => countryCasesMap[d.properties.name] ? "#32CD32" : "#AAAAAA") // Fallback color if no data
                    .on("mouseover", function(d) {
                        d3.select(this).style("fill", "#FFD700"); // Highlight color
                        showCountryName(d.properties.name, countryCasesMap[d.properties.name] || 'No data');
                    })
                    .on("mouseout", function(d) {
                        d3.select(this).style("fill", d => countryCasesMap[d.properties.name] ? "#32CD32" : "#AAAAAA"); // Reset color
                        hideCountryName();
                    });
            });
        }



        function showCountryName(name, data) {
            const displayData = data || 'No data'; // Ensure there's a fallback if no data is available
            svg.append("text")
                .attr("class", "countryName")
                .attr("x", 20)
                .attr("y", 35)
                .text(`${name}: ${displayData}`) // Display the country name and its data
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

        let rotationTimer; // 定义一个变量来跟踪旋转计时器

        function enableRotation() {
            if (rotationTimer) {
                rotationTimer.stop();
            }

            let lastTime = Date.now(); // 初始化上一次旋转的时间

            rotationTimer = d3.timer(function() {
                if (!isDragging) {
                    const now = Date.now(); // 获取当前时间
                    const delta = now - lastTime; // 计算时间差
                    lastTime = now; // 更新上一次旋转的时间

                    const rotate = projection.rotate();
                    const rotationSpeed = 0.02; // 控制旋转速度，可根据需要调整
                    projection.rotate([rotate[0] + rotationSpeed * delta, rotate[1]]);
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
        // Function to update the bar chart
        function updateBarChart(selectedYear) {
            // Filter and sort data
            const sortedData = csvData.filter(d => d.Year == selectedYear)
                .sort((a, b) => b["GDP per capita, PPP (constant 2017 international $)"] - a["GDP per capita, PPP (constant 2017 international $)"])
                .slice(0, 15); // Take top 15

            // Update scales
            const x = d3.scaleLinear()
                .range([margin.left, barChartWidth - margin.right])
                .domain([0, d3.max(sortedData, d => d["GDP per capita, PPP (constant 2017 international $)"])]);

            const y = d3.scaleBand()
                .range([margin.top, barChartHeight - margin.bottom])
                .domain(sortedData.map(d => d.Entity))
                .padding(0.1);

            // Remove old bars
            barSvg.selectAll(".bar").remove();

            // Remove old axes
            barSvg.selectAll("g.axis").remove();

            // Draw bars
            barSvg.selectAll(".bar")
                .data(sortedData)
                .enter().append("rect")
                .attr("class", "bar")
                .attr("x", x(0))
                .attr("y", d => y(d.Entity))
                .attr("width", d => x(d["GDP per capita, PPP (constant 2017 international $)"]) - x(0))
                .attr("height", y.bandwidth());

            // Add axes
            barSvg.append("g")
                .attr("class", "axis")
                .attr("transform", "translate(0," + (barChartHeight - margin.bottom) + ")")
                .call(d3.axisBottom(x).ticks(5).tickFormat(d3.format("~s")));

            barSvg.append("g")
                .attr("class", "axis")
                .attr("transform", "translate(" + margin.left + ",0)")
                .call(d3.axisLeft(y));

            // Add country names and GDP data
            barSvg.selectAll(".text")
                .data(sortedData)
                .enter().append("text")
                .attr("class", "label")
                .attr("x", d => x(d["GDP per capita, PPP (constant 2017 international $)"]) - 3)
                .attr("y", d => y(d.Entity) + y.bandwidth() / 2)
                .attr("dy", ".35em")
                .text(d => `${d.Entity}: ${d["GDP per capita, PPP (constant 2017 international $)"]}`)
                .attr("text-anchor", "end")
                .style("fill", "black");
        }

        // Call updateBarChart when the year is selected
        d3.select("#yearSelector").on("change", function() {
            updateMapForYear(this.value);
            updateBarChart(this.value);
        });

        // Initialize the bar chart with the first available year
        updateBarChart(years[0]);
    </script>
</body>
</html>
