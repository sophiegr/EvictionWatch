<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";
    // Define your data
    const data = [
        { neighborhood: "Allston", own_occ_rate_diff: -0.06 },
        { neighborhood: "Beacon Hill", own_occ_rate_diff: -0.04 },
        { neighborhood: "Brighton", own_occ_rate_diff: -0.12 },
        { neighborhood: "Central Boston", own_occ_rate_diff: -0.05 },
        { neighborhood: "Charlestown", own_occ_rate_diff: -0.03 },
        { neighborhood: "Chinatown", own_occ_rate_diff: 0.02 },
        { neighborhood: "Dorchester", own_occ_rate_diff: -0.01 },
        { neighborhood: "East Boston", own_occ_rate_diff: -0.06 },
        { neighborhood: "Fenway", own_occ_rate_diff: -0.06 },
        { neighborhood: "Hyde Park", own_occ_rate_diff: 0.0 },
        { neighborhood: "Jamaica Plain", own_occ_rate_diff: -0.03 },
        { neighborhood: "Longwood", own_occ_rate_diff: -0.1 },
        { neighborhood: "Mattapan", own_occ_rate_diff: -0.01 },
        { neighborhood: "North End", own_occ_rate_diff: -0.1 },
        { neighborhood: "Roslindale", own_occ_rate_diff: -0.05 },
        { neighborhood: "Roxbury", own_occ_rate_diff: 0.02 },
        { neighborhood: "South Boston", own_occ_rate_diff: -0.1 },
        { neighborhood: "South Boston Waterfront", own_occ_rate_diff: 0.05 },
        { neighborhood: "South End", own_occ_rate_diff: -0.09 },
        { neighborhood: "West End", own_occ_rate_diff: -0.15 },
        { neighborhood: "West Roxbury", own_occ_rate_diff: -0.01 }
    ];

    // Set up the SVG canvas dimensions
    const margin = { top: 20, right: 30, bottom: 50, left: 175 };
    const width = 800 - margin.left - margin.right;
    const height = 500 - margin.top - margin.bottom;

    onMount(() => {

    // Append the SVG object to the body of the page
    const svg = d3.select("#my_dataviz2")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

   // X scale
    const x = d3.scaleLinear()
        .domain([
            d3.min(data, d => d.own_occ_rate_diff) - 0.01, // 10% padding on the left
            d3.max(data, d => d.own_occ_rate_diff) + 0.01  // 10% padding on the right
        ])
        .nice()
        .range([0, width]);

    // Y scale
    const y = d3.scaleBand()
        .domain(data.map(d => d.neighborhood))
        .range([0, height])
        .padding(0.1);
    
    // Add X axis
    svg.append("g")
        .attr("class", "axis-x-2")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(x));
    
    // Add Y axis
    svg.append("g")
        .attr("class", "axis-y-2")
        .call(d3.axisLeft(y));

    // Add X axis label
    svg.append("text")
        .attr("transform", `translate(${width / 2},${height + margin.top + 20})`)
        .style("text-anchor", "middle")
        .text("Change in Owner Occupancy Rate");

    // Add Y axis label
    svg.append("text")
        .attr("transform", "rotate(-90)")
        .attr("y", 0 - margin.left)
        .attr("x", 0 - (height / 2))
        .attr("dy", "1em")
        .style("text-anchor", "middle")
        .text("Neighborhoods");
    
    // Add bars
    svg.selectAll(".bar")
        .data(data)
        .enter().append("rect")
        .attr("class", "bar2")
        .attr("y", d => y(d.neighborhood))
        .attr("height", y.bandwidth())
        .attr("x", d => x(Math.min(0, d.own_occ_rate_diff)))
        .attr("width", d => Math.abs(x(d.own_occ_rate_diff) - x(0)))
        .on("mouseover", function (event, d) {
                tooltip.style("opacity", 1)
                    .html(`Value: ${d.own_occ_rate_diff}`)
                    .style("left", (event.pageX + 10) + "px")
                    .style("top", (event.pageY - 20) + "px");
            })
        .on("mouseout", function () {
            tooltip.style("opacity", 0);
        });
    
    // Tooltip
    const tooltip = d3.select("#my_dataviz2")
            .append("div")
            .style("opacity", 0)
            .attr("class", "tooltip");

    })

</script>

<!-- svelte-ignore css_unused_selector -->
<!-- <body> -->
    <div id="my_dataviz2"></div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Sans:ital@0;1&display=swap');
    /* Add your CSS styles here */
    :global(body) {
        font-family: "Noto Sans", sans-serif;
    }
    :global(.bar2) {
        fill: rgb(128, 63, 128);
    }

    :global(.bar2:hover) {
        fill: steelblue;
    }

    :global(.axis-x-2, .axis-y-2) {
        font-family: "Noto Sans", sans-serif;
        font-size: 14px;
    }

    :global(.axis-x-2 text, .axis-y-2 text) {
        font-family: "Noto Sans", sans-serif;
        fill: black;
    }

    :global(.axis-x-2 line, .axis-y-2 line) {
        stroke: #ccc;
    }

    :global(.axis-x-2 path, .axis-y-2 path) {
        display: none;
    }

    :global(.tooltip) {
        position: absolute;
        text-align: center;
        padding: 5px;
        font: 12px sans-serif;
        background: #ffffff;
        border: 1px solid #ddd;
        pointer-events: none;
        opacity: 0;
    }
</style>