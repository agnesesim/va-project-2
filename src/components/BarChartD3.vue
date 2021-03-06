<template>
    <div>
        <svg :id="this.idSvg" width='100%' height='400'>
            </svg>
    </div>
</template>

<script>
    const d3 = require('d3');
    import $ from 'jquery';

    function BarChart(data, {
        x = (d, i) => i, // given d in data, returns the (ordinal) x-value
        y = d => d, // given d in data, returns the (quantitative) y-value
        marginTop = 20, // the top margin, in pixels
        marginRight = 0, // the right margin, in pixels
        marginBottom = 30, // the bottom margin, in pixels
        marginLeft = 20, // the left margin, in pixels
        width = 640, // the outer width of the chart, in pixels
        height = 400, // the outer height of the chart, in pixels
        xDomain, // an array of (ordinal) x-values
        xRange = [marginLeft, width - marginRight], // [left, right]
        yType = d3.scaleLinear, // y-scale type
        yDomain, // [ymin, ymax]
        yRange = [height - marginBottom, marginTop], // [bottom, top]
        xPadding = 0.1, // amount of x-range to reserve to separate bars
        yFormat, // a format specifier string for the y-axis
        yLabel, // a label for the y-axis
        color = "currentColor" // bar fill color
        } = {},
        id
        ) {
        
        $('#'+id).empty();
        const svg = d3.select('#'+id);
        width = svg.node().getBoundingClientRect().width;
        xRange = [marginLeft, width - marginRight]

        // Compute values.
        const X = d3.map(data, x);
        const Y = d3.map(data, y);

        // Compute default domains, and unique the x-domain.
        if (xDomain === undefined) xDomain = X;
        if (yDomain === undefined) yDomain = [0, d3.max(Y)];
        xDomain = new d3.InternSet(xDomain);

        // Omit any data not present in the x-domain.
        const I = d3.range(X.length).filter(i => xDomain.has(X[i]));

        // Construct scales, axes, and formats.
        const xScale = d3.scaleBand(xDomain, xRange).padding(xPadding);
        const yScale = yType(yDomain, yRange);
        const xAxis = d3.axisBottom(xScale).tickSizeOuter(0);
        const yAxis = d3.axisLeft(yScale).ticks(height / 40, yFormat);
        
        svg.append("g")
            .attr("transform", `translate(${marginLeft},0)`)
            .call(yAxis)
            .call(g => g.select(".domain").remove())
            .call(g => g.selectAll(".tick line").clone()
                .attr("x2", width - marginLeft - marginRight)
                .attr("stroke-opacity", 0.1))
            .call(g => g.append("text")
                .attr("x", -marginLeft)
                .attr("y", 10)
                .attr("fill", "currentColor")
                .attr("text-anchor", "start")
                .text(yLabel));

        svg.append("g")
            .attr("fill", color)
            .selectAll("rect")
            .data(I)
            .join("rect")
            .attr("x", i => xScale(X[i]) +  xScale.bandwidth()/2 - 12)
            .attr("y", i => yScale(Y[i]))
            .attr("rx", 10)
            .attr("height", i => yScale(0) - yScale(Y[i]))
            .attr("width", 25);

        svg.append("g")
            .attr("transform", `translate(0,${height - marginBottom})`)
            .call(xAxis);

        return svg.node();
    }

    export default {
        name: "BarChartD3",
        props:{
            data_source: Array,
            options: Object,
            idSvg: {
                type: String,
                default: 'idDiDefault'
            }
        },
        data: function(){
            return {
                chart: BarChart(this.data_source, this.options, this.idSvg),
            }
        },
        mounted(){
          this.refreshPlot();
        },
        methods:{
          refreshPlot: function(){
            this.chart = BarChart(this.data_source, this.options, this.idSvg)
          }
        },
        watch:{
           data_source: function(){
               this.refreshPlot();
           }
        }
    }
</script>
