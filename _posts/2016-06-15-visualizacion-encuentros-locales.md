---
layout: post
title: "Visualización de Encuentros Locales"
date: 2016-06-15
---
Datos actualizados al 21 de junio de 2016.

## Número de cabildos vs. ingreso medio
<div class="graph_scatter"></div>

## Distribución geográfica de cabildos
<div class="graph_map"></div>

<style>
[class^="label"] {
	font-size: 20%;
}

.comunas {
	fill: none;
	stroke: #000;
	stroke-width: 0.2px;
}

.legend {
	font-family: Helvetica;
	font-size: 75%;
}

div.tooltip {
	position: absolute;
	text-align: center;
	width: 150px;
	height: 25px;
	line-height: 25px;
	white-space: nowrap;
	font-size: 75%;
	font-family: Helvetica;
	border-radius: 4px;  
	background: #FFF;
}

[class^="graph"] {
	text-align:center;
}

.axis path,
.axis line {
  fill: none;
  stroke: #000;
  shape-rendering: crispEdges;
}

.dot {
	stroke: #000;
	fill: none;
}

.tooltip_scatter {
	position: absolute;
	text-align: left;
	width: 150px;
	height: 25px;
	line-height: 25px;
	white-space: nowrap;
	font-size: 100%;
	font-family: Helvetica;
	border-radius: 4px;
	background: #FFF;
}

.graph_scatter {
	font: 10px sans-serif;
}

.label {
	font: 10px sans-serif;
}
</style>

<script src="//d3js.org/d3.v3.min.js"></script>
<script src="//d3js.org/queue.v1.min.js"></script>
<script src="//d3js.org/topojson.v1.min.js"></script>
<script>


var width = 600, height = 800;

var cabildosByComuna = d3.map();

var quantile = d3.scale.quantile()
	.range(["rgb(247,251,255)", "rgb(222,235,247)", "rgb(198,219,239)", "rgb(158,202,225)", "rgb(107,174,214)", "rgb(66,146,198)", "rgb(33,113,181)", "rgb(8,81,156)", "rgb(8,48,107)"]);
	

var projection = d3.geo.mercator()
	.scale(1)
	.translate([0, 0]);

var path = d3.geo.path()
	.projection(projection);

var div_map = d3.select(".graph_map").append("div")
	.attr("class", "tooltip")
	.style("opacity", 0);

var svg_map = d3.select(".graph_map").append("svg")
	.attr("width", width)
	.attr("height", height);

queue()
	.defer(d3.json, "https://cdn.rawgit.com/elaval/d3-comunas-cl/master/data/comunas.json")
	.defer(d3.csv, "{{ site.github.url }}/data/cabildos-comunas.csv",
	function(d) {cabildosByComuna.set(d.comuna, +d.cabildo_per_capita); })
	.await(ready);

function ready(error, chile) {
	if (error) throw error;
	var cl_comunas = topojson.feature(chile, chile.objects.cl_comunas);
	
	var b = path.bounds(cl_comunas),
		s = .95 / Math.max((b[1][0] - b[0][0]) / width, (b[1][1] - b[0][1]) / height),
		t = [(width - s * (b[1][0] + b[0][0])) / 2, (height - s * (b[1][1] + b[0][1])) / 2];

	projection
		.scale(s)
		.translate(t);
	
	quantile.domain(cabildosByComuna.values());
	
	svg_map.append("g")
		.attr("class", "comunas")
	.selectAll("path")
		.data(cl_comunas.features)
	.enter().append("path")
		.attr("fill", function(d) { return quantile(cabildosByComuna.get(d.id)); })
		.attr("class", function(d) { return "comuna " + d.properties.name; })
	.attr("d", path)
	
	.on("mouseover", function(d) {
		d3.select(this).transition().duration(300).style("opacity", 1);
		div_map.transition().duration(100)
		.style("opacity", 1)
		div_map.text(d.properties.name + " : " + Math.round(cabildosByComuna.get(d.id)*100)/100)
		.style("left", (d3.event.pageX) + "px")
		.style("top", (d3.event.pageY - 30) + "px");
	})
	.on("mouseout", function() {
		d3.select(this)
		.transition().duration(300)
		.style("opacity", 0.8);
		div_map.transition().duration(300)
		.style("opacity", 0);
	});
	
		var legend = svg_map.selectAll("g.legend")
		.data(quantile.quantiles())
		.enter().append("g")
		.attr("class", "legend");

		var ls_w = 20, ls_h = 20;

		legend.append("rect")
		.attr("x", 20)
		.attr("y", function(d, i){ return height - (i*ls_h) - 2*ls_h;})
		.attr("width", ls_w)
		.attr("height", ls_h)
		.style("fill", function(d, i) { return quantile(d); })
		.style("opacity", 0.8);

		legend.append("text")
		.attr("x", 50)
		.attr("y", function(d, i){ return height - (i*ls_h) - ls_h - 4;})
		.text(function(d, i){ return Math.round(d*100)/100; });
}

var margin = {top: 20, right: 20, bottom: 30, left: 40},
	width_scatter = 600 - margin.left - margin.right,
	height_scatter = 300 - margin.top - margin.bottom;

var xValue = function(d) { return d.ytotcor;},
	xScale = d3.scale.linear().range([0, width_scatter]),
	xMap = function(d) { return xScale(xValue(d));},
	xAxis = d3.svg.axis().scale(xScale).orient("bottom");

var yValue = function(d) { return d.numero_cabildos;},
	yScale = d3.scale.linear().range([height_scatter, 0]),
	yMap = function(d) { return yScale(yValue(d));},
	yAxis = d3.svg.axis().scale(yScale).orient("left");

var svg_scatter = d3.select(".graph_scatter").append("svg")
	.attr("width", width_scatter + margin.left + margin.right)
	.attr("height", height_scatter + margin.top + margin.bottom)
	.append("g")
	.attr("transform", "translate(" + margin.left + "," + margin.top + ")");

var div_scatter = d3.select(".graph_scatter").append("div")
	.attr("class", "tooltip_scatter")
	.style("opacity", 0);

d3.csv("{{ site.github.url }}/data/cabildos-comunas.csv", function(error, data) {

	data.forEach(function(d) {
		d.numero_cabildos = +d.numero_cabildos;
		d.ytotcor = +d.ytotcor;
	});

	xScale.domain([d3.min(data, xValue)-1, d3.max(data, xValue)+1]);
	yScale.domain([d3.min(data, yValue)-1, d3.max(data, yValue)+1]);

	svg_scatter.append("g")
		.attr("class", "x axis")
		.attr("transform", "translate(0," + height_scatter + ")")
		.call(xAxis)
	.append("text")
		.attr("class", "label")
		.attr("x", width_scatter)
		.attr("y", -6)
		.style("text-anchor", "end")
		.text("Ln(Ingreso)");

	svg_scatter.append("g")
		.attr("class", "y axis")
		.call(yAxis)
	.append("text")
		.attr("class", "label")
		.attr("transform", "rotate(-90)")
		.attr("y", 6)
		.attr("dy", ".71em")
		.style("text-anchor", "end")
		.text("Número de cabildos");

	svg_scatter.selectAll(".dot")
		.data(data)
	.enter().append("circle")
		.filter(function(d) { return !isNaN(d.ytotcor) })
		.attr("class", "dot")
		.attr("r", 3)
		.attr("cx", xMap)
		.attr("cy", yMap)
		.on("mouseover", function(d) {
			div_scatter.transition()
				.duration(100)
				.style("opacity", .9);
			div_scatter.html(d.nombre_comuna)
				.style("left", (d3.event.pageX + 5) + "px")
				.style("top", (d3.event.pageY - 28) + "px");
		})
		.on("mouseout", function(d) {
			div_scatter.transition()
				.duration(300)
				.style("opacity", 0);
		});
});

</script>