<!DOCTYPE html>
<html>
    <head>
        <style>
          .bar {
            fill: steelblue;
        }
        
        .bar:hover {
            fill: orange;
        }
        
        .axis path,
        .axis line {
            fill: none;
            stroke: #000;
            shape-rendering: crispEdges;
        }
        
        .axis text {
            font-family: sans-serif;
            font-size: 11px;
        }
        
        .axis-label {
            font-size: 14px;
            text-anchor: middle;
        }
        #lineGraphContainer {
    position: absolute;
    top: 110px; /* Adjust as needed */
    right: 250px; /* Adjust as needed */
    width: 300px; /* Maintain or adjust size as needed */
    height: 200px; /* Maintain or adjust size as needed */
}
        </style>
    </head>
<body>
    <!-- Dropdown for selecting a year -->
    <label for="yearSelector" style="font-size: 20px;">Which year are you interested in?</label>
    <select id="yearSelector" style="font-size: 15px;" ></select>
    <div id="lineGraphContainer"></div>
    <div id="visualizationContainer" style="display: flex; justify-content: space-between; align-items: start;">
        <div id="globeContainer" style="flex-grow: 1;">
            <!-- Globe SVG and related elements here -->
        </div>
        <div id="lineGraphContainer" style="width: 300px; height: 200px;">
            <!-- Line graph will be rendered here by Plotly -->
        </div>
    </div>
    
    <svg>

    </svg>
    <!-- D3.js and TopoJSON for map rendering -->
    <script src="https://d3js.org/d3.v4.min.js"></script>
    <script src="https://d3js.org/topojson.v1.min.js"></script>
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script> <!-- Plotly.js for line graph -->
    <script>
        // Place this at the top of your D3.js script, after setting up the main SVG
// Define axes here as well

        const width = 1200;
        const height = 700;
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
        const margin = { top: 20, right: 240, bottom: 70, left: 70 },
              barChartWidth = 1000 ,
              barChartHeight = 500;

        const xScale = d3.scaleLinear()
            .range([0, barChartWidth]);
            
        const yScale = d3.scaleBand()
            .range([barChartHeight, 0])
            .padding(0.1);
            const svgContainer = d3.select('svg');
            function generateLineGraphData(countryName) {
        const countryData = csvData.filter(d => d.Entity === countryName);
        const years = countryData.map(d => +d.Year);
        const gdpPerCapita = countryData.map(d => +d["GDP per capita, PPP (constant 2017 international $)"]);

        return {
            years: years,
            gdpPerCapita: gdpPerCapita
        };
    }
    // Example function to add a highlighted point on the line graph for the selected year
    function updateGraphWithHighlightedYear(countryName, selectedYear) {
    const lineGraphData = generateLineGraphData(countryName);

    // Debugging: Log the data to verify it's as expected
    console.log("Line Graph Data:", lineGraphData);
    
    const index = lineGraphData.years.findIndex(year => year === selectedYear);
    if (index === -1) {
        console.error("Selected year not found in the data");
        return;
    }

    const highlightX = [lineGraphData.years[index]];
    const highlightY = [lineGraphData.gdpPerCapita[index]];

    // Debugging: Log the highlight data
    console.log("Highlight Data:", highlightX, highlightY);

    const trace1 = {
        x: lineGraphData.years,
        y: lineGraphData.gdpPerCapita,
        type: 'scatter',
        mode: 'lines+markers',
        marker: { color: 'blue' },
    };

    const trace2 = {
        x: highlightX,
        y: highlightY,
        type: 'scatter',
        mode: 'markers',
        marker: { color: 'red', size: 14 },
    };

    const layout = {
        title: `${countryName} - GDP Per Capita Over Time`,
    };

    Plotly.newPlot('lineGraphContainer', [trace1, trace2], layout);
}


    // Function to create a line graph for a given country using Plotly.js
    function createLineGraph(countryName) {
    const lineGraphData = generateLineGraphData(countryName);

    const data = [{
        x: lineGraphData.years,
        y: lineGraphData.gdpPerCapita,
        type: 'scatter',
        mode: 'lines+markers',
        marker: { color: 'blue' },
        name: 'GDP per capita'
    }];

    const layout = {
        title: `${countryName} - GDP Per Capita Over Time`,
        xaxis: { title: 'Year' },
        yaxis: { title: 'GDP per capita' },
        width: 500, // Adjust width as needed
        height: 500, // Adjust height as needed
        margin: { // Adjust margins as needed
            l: 50,
            r: 50,
            b: 50,
            t: 50,
            pad: 4
        },
    };

    Plotly.newPlot('lineGraphContainer', data, layout);
}

// Add a title to the SVG
svgContainer.append("text")
  .attr("x", width / 2  ) // This will position the text in the center of the SVG
  .attr("y", margin.top + 20) // This positions the text from the top of the SVG
  .attr("text-anchor", "middle") // This centers the text (horizontally)
  .style("font-size", "24px") // Change the font size as needed
  .style("font-weight", "bold") // Optional: makes the text bold
  .text("GDP Per Capita Over Time 1990 - 2021"); // Replace with your actual title
            
    const barSvg = d3.select('body').append('svg')
    .attr('width', barChartWidth + margin.left + margin.right)
    .attr('height', barChartHeight + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`);


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
             d3.selectAll(".segment")
            .on("mouseover", function(d) {
                const countryName = d.properties.name;
                const selectedYear = parseInt(document.getElementById("yearSelector").value);
                console.log("Hovered country:", countryName);
                createLineGraph(countryName); // Generate and display line graph
                highlightYearOnGraph(selectedYear);
            })
            .on("mouseout", function(d) {
                console.log("Mouseout");
                d3.select('#lineGraphContainer').selectAll("*").remove(); // Clear line graph
            });
            d3.selectAll(".country").on("mouseover", function(event, d) {
    const countryName = d.properties.name;
    const selectedYear = parseInt(document.getElementById("yearSelector").value);
    updateGraphWithHighlightedYear(countryName, selectedYear);
});

    // First, generate or update the line graph for the hovered country
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
                    const countryName = d.properties.name;
                    createLineGraph(countryName); // This line was corrected
                })
                .on("mouseout", function(d) {
                    d3.select(this).style("fill", d => countryCasesMap[d.properties.name] ? "#32CD32" : "#AAAAAA"); // Reset color
                    // Clear the line graph container
                    d3.select('#lineGraphContainer').html(''); // This line ensures the line graph is cleared on mouseout
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
                    const rotationSpeed = 0.005; // 控制旋转速度，可根据需要调整
                    projection.rotate([rotate[0] + rotationSpeed * delta, rotate[1]]);
                    svg.selectAll("path").attr("d", path);
                }
            });
        }

        // Function to update the line graph based on the country name

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

        // Function to update the bar chart
       // Assuming that your sortedData is correctly sorted and contains the data for the selected year
function updateBarChart(selectedYear) {
    // Filter and sort data
    const sortedData = csvData.filter(d => d.Year == selectedYear)
        .sort((a, b) => b["GDP per capita, PPP (constant 2017 international $)"] - a["GDP per capita, PPP (constant 2017 international $)"])
        .slice(0, 15); // Take top 15

        const maxBarHeight = barChartHeight / sortedData.length;

const yScale = d3.scaleBand()
    .rangeRound([0, barChartHeight]) // The range is the total height available
    .paddingInner(0.1) // Adjust padding to manage spacing between bars
    .domain(sortedData.map(d => d.Entity));

// Now you update the bar height when appending the bars using the yScale
barSvg.selectAll(".bar")
    .data(sortedData)
    .enter().append("rect")
    .attr("class", "bar")
    .attr("x", 0)
    .attr("y", d => yScale(d.Entity))
    .attr("width", d => xScale(+d["GDP per capita, PPP (constant 2017 international $)"]))
    .attr("height", yScale.bandwidth()); // This uses the scale's computed bandwidth
    barSvg.selectAll(".bar")
  // ... (existing code for creating bars)
// Update the SVG height if necessary based on the new calculations
const totalHeightRequired = maxBarHeight * sortedData.length;
if (totalHeightRequired > barChartHeight) {
    // Increase the height of the SVG to accommodate all bars
    d3.select('svg')
        .attr('height', totalHeightRequired + margin.top + margin.bottom);
}
    // Update scales
    xScale.domain([0, d3.max(sortedData, d => +d["GDP per capita, PPP (constant 2017 international $)"])]);
    yScale.domain(sortedData.map(d => d.Entity));
    barSvg.selectAll('*').remove();
    const g = barSvg.append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);
    // Remove old bars and labels
    barSvg.selectAll(".bar").remove();
    barSvg.selectAll(".label").remove();
    barSvg.selectAll(".value").remove();
    barSvg.selectAll(".axis").remove();

    barSvg.selectAll(".bar")
    .on("mouseover", function(d) {
        // Call the highlight function with the country name
        highlightCountryOnGlobe(d.Entity);
    })
    .on("mouseout", function() {
        // Reset the globe visualization when not hovering
        drawGlobe();
    });
    // Draw bars
    g.selectAll('.bar')
        .data(sortedData)
        .enter().append('rect')
        .attr('class', 'bar')
        .attr('x', 40)
        .attr('y', d => yScale(d.Entity))
        .attr('width', d => xScale(+d["GDP per capita, PPP (constant 2017 international $)"]))
        .attr('height', yScale.bandwidth());


    // Add x and y axes
    g.append('g')
        .attr('transform', `translate(40,${barChartHeight})`)
        .call(d3.axisBottom(xScale));

    // Draw the Y axis

    // Append country names to the left of the bars
    barSvg.selectAll(".label")
        .data(sortedData)
        .enter().append("text")
        .attr("class", "label")
        .attr("x", 100)
        .attr("y", d => yScale(d.Entity) + yScale.bandwidth() / 2+ 15)
        .attr("dy", ".35em")
        .attr("text-anchor",    "end")
        .text(d => d.Entity);

    // Append values at the end of the bars
    barSvg.selectAll(".value")
        .data(sortedData)
        .enter().append("text")
        .attr("class", "value")
        .attr("x", d => xScale(d["GDP per capita, PPP (constant 2017 international $)"]) + 120)
        .attr("y", d => yScale(d.Entity) + yScale.bandwidth() / 2 + 20 )
        .attr("dy", ".35em")
        .attr("text-anchor", "start")
        .text(d => d["GDP per capita, PPP (constant 2017 international $)"]);
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
