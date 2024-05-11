<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";
    // Define your data
    const data = [
        { neighborhood: "Allston", corp_own_rate_diff: 0.21 },
        { neighborhood: "Beacon Hill", corp_own_rate_diff: 0.23 },
        { neighborhood: "Brighton", corp_own_rate_diff: 0.17 },
        { neighborhood: "Central Boston", corp_own_rate_diff: 0.19 },
        { neighborhood: "Charlestown", corp_own_rate_diff: 0.14 },
        { neighborhood: "Chinatown", corp_own_rate_diff: 0.14 },
        { neighborhood: "Dorchester", corp_own_rate_diff: 0.14 },
        { neighborhood: "East Boston", corp_own_rate_diff: 0.19 },
        { neighborhood: "Fenway", corp_own_rate_diff: 0.26 },
        { neighborhood: "Hyde Park", corp_own_rate_diff: 0.1 },
        { neighborhood: "Jamaica Plain", corp_own_rate_diff: 0.14 },
        { neighborhood: "Longwood", corp_own_rate_diff: 0.21 },
        { neighborhood: "Mattapan", corp_own_rate_diff: 0.1 },
        { neighborhood: "North End", corp_own_rate_diff: 0.2 },
        { neighborhood: "Roslindale", corp_own_rate_diff: 0.1 },
        { neighborhood: "Roxbury", corp_own_rate_diff: 0.18 },
        { neighborhood: "South Boston", corp_own_rate_diff: 0.17 },
        { neighborhood: "South Boston Waterfront", corp_own_rate_diff: 0.29 },
        { neighborhood: "South End", corp_own_rate_diff: 0.18 },
        { neighborhood: "West End", corp_own_rate_diff: 0.22 },
        { neighborhood: "West Roxbury", corp_own_rate_diff: 0.1 }
    ];

    // Set up the SVG canvas dimensions
    const margin = { top: 20, right: 30, bottom: 50, left: 175 };
    const width = 800 - margin.left - margin.right;
    const height = 500 - margin.top - margin.bottom;

    onMount(() => {

        // Append the SVG object to the body of the page
        const svg = d3.select("#my_dataviz")
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);
        

        // X scale
        const x = d3.scaleLinear()
            .domain([0, d3.max(data, d => d.corp_own_rate_diff)+0.02])
            .range([0, width]);

        // Y scale
        const y = d3.scaleBand()
            .domain(data.map(d => d.neighborhood))
            .range([0, height])
            .padding(0.1);

        // Add X axis
        svg.append("g")
            .attr("class", "axis-x")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(x));

        // Add Y axis
        svg.append("g")
            .attr("class", "axis-y")
            .call(d3.axisLeft(y));

        // Add X axis label
        svg.append("text")
            .attr("transform", `translate(${width / 2},${height + margin.top + 20})`)
            .style("text-anchor", "middle")
            .text("Change in Corporate Ownership Rate");

        // Add Y axis label
        svg.append("text")
            .attr("transform", "rotate(-90)")
            .attr("y", 0 - margin.left)
            .attr("x", 0 - (height / 2))
            .attr("dy", "1em")
            .style("text-anchor", "middle")
            .text("Neighborhoods");

        // Add bars
        svg.selectAll("mybar")
            .data(data)
            .enter()
            .append("rect")
            .attr("class", "bar")
            .attr("x", 0)
            .attr("y", d => y(d.neighborhood))
            .attr("width", d => x(d.corp_own_rate_diff))
            .attr("height", y.bandwidth())
            .on("mouseover", function (event, d) {
                tooltip.style("opacity", 1)
                    .html(`Value: ${d.corp_own_rate_diff}`)
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
<div id="my_dataviz"></div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Sans:ital@0;1&display=swap');
    /* Add your CSS styles here */
    :global(.bar) {
        fill: orange;
    }

    :global(.bar:hover) {
        fill: steelblue;
    }

    :global(.axis-x, .axis-y) {
        font-family: "Noto Sans", sans-serif;
        font-size: 14px;
    }

    :global(.axis-x text, .axis-y text) {
        font-family: "Noto Sans", sans-serif;
        fill: black;
    }

    :global(.axis-x line, .axis-y line) {
        stroke: #ccc;
    }

    :global(.axis-x path, .axis-y path) {
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