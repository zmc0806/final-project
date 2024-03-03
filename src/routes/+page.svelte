<!DOCTYPE html>
<html>
<head>
    <title>World GDP Visualization</title>
    <script src="https://d3js.org/d3.v4.min.js"></script>
    <style>
        .visualization-container {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    
    <!-- Dropdown for selecting a year -->
    <label for="yearSelector">Select Year:</label>
    <select id="yearSelector"></select>
    
    <!-- Container for the globe and bar chart -->
    <div class="visualization-container">
        <div id="barChart" style="width: 400px; height: 500px; float: left;"></div>
        <svg id="globe" width="960" height="500" style="float: right;"></svg>
    </div>

    
    <!-- D3.js and TopoJSON for map rendering -->
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

            // Initialize the visualization with the first available year
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
    
    // Update the visualization including the bar chart
    updateVisualization(selectedYear);
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

        function updateVisualization(selectedYear) {
    // Filter data based on the selected year and whether we're looking at GDP
    const filteredData = csvData.filter(d => d.Year == selectedYear);
    const dataForMap = {};
    let dataForBarChart = [];

    filteredData.forEach(d => {
        dataForMap[d.Entity] = +d["GDP per capita"]; // Assuming GDP data
        dataForBarChart.push({
            country: d.Entity,
            value: +d["GDP per capita"]
        });
    });

    // Sort the data based on the GDP per capita value in descending order
    dataForBarChart.sort((a, b) => b.value - a.value);

    // Take the top 20 countries for the bar chart
    dataForBarChart = dataForBarChart.slice(0, 20);

    drawGlobe(dataForMap);
    drawBarChart(dataForBarChart);
}


function drawBarChart(data) {
    // Clear the previous bar chart
    d3.select("#barChart").selectAll("*").remove();

    const margin = {top: 20, right: 20, bottom: 30, left: 40},
        width = 400 - margin.left - margin.right,
        height = 500 - margin.top - margin.bottom;

    const x = d3.scaleLinear().range([0, width]);
    const y = d3.scaleBand().range([height, 0]).padding(0.1);

    const chart = d3.select("#barChart").append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

    // Sort the data in descending order based on GDP per capita value
    data.sort((a, b) => b.value - a.value);

    x.domain([0, d3.max(data, d => d.value)]);
    y.domain(data.map(d => d.country));

    chart.selectAll(".bar")
        .data(data)
        .enter().append("rect")
        .attr("class", "bar")
        .attr("x", 0) // Start bars from the left edge
        .attr("y", d => y(d.country))
        .attr("width", d => x(d.value))
        .attr("height", y.bandwidth())
        .style("fill", "#69b3a2");

    chart.append("g")
        .attr("class", "x axis")
        .call(d3.axisBottom(x));

    chart.append("g")
        .attr("class", "y axis")
        .call(d3.axisLeft(y));
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

        function loadRegionMap(regionFile) {
            if (!regionFile || regionFile === "") return; // Exit if no region selected

            d3.json(regionFile, function(error, region) {
                if (error) {
                    console.error("Error loading the region map:", error);
                    return; // Exit if there's an error loading the file
                }

                regionSvg.selectAll("*").remove(); // Clear existing map

                // Assuming the GeoJSON structure has a 'features' array directly under 'region'
                // Adjust the feature extraction as per your GeoJSON structure
                const features = region.features || (region.objects ? topojson.feature(region, region.objects.countries).features : []);

                regionSvg.selectAll(".country")
                    .data(features)
                    .enter().append("path")
                    .attr("class", "country")
                    .attr("d", regionPath)
                    .style("fill", "#ccc"); // Adjust style as needed
            });
        }

        d3.select("#regionSelector").on("change", function() {
            const selectedRegion = this.value;
            const regionFile = regionFiles[selectedRegion];
            if (regionFile && regionFile !== "") {
                loadRegionMap(regionFile);
            } else {
                console.log("No region file found for:", selectedRegion);
            }
        });

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
            
    </script>

</body>
</html>
