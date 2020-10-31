---
layout: post
title: Math Skill Graph v01
author: sqrtnull
date: 2020-10-23
category: d3
---

<style type="text/css">
svg.graph {
    background-color: white;
    cursor: default;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    -o-user-select: none;
    user-select: none;
}
circle.node {
    stroke-width: 1.5px;
}
line.link {
  stroke-width: 2px;
  stroke: grey;
}
</style>

<script type="text/javascript">
const width = 1024;
const height = 1024;
const colors = d3.scaleOrdinal(d3.schemeCategory10);
const svg = d3.select('body')
    .append('div')
    .append('svg')
    .attr('viewBox', '0 0 ' + width + ' ' + height)
    .classed('graph', true)
    .on('contextmenu', () => { d3.event.preventDefault(); });

let max_difficulty=1;

let nodes = [
];
let links = [
];

let data = {
    Axiom: [
    { statement: 'a', link: '' },
    { statement: 'b', link: '' }
    ],
    Theorem: [
    { statement: 'c', link: '', proofs: [
        { statement: 'e', link: '', assumptions: [0] }
    ] },
    { statement: 'd', link: '', proofs: [
        { statement: 'f', link: '', assumptions: [0,1] }
    ] }
    ]
};

const force = d3.forceSimulation()
    .force('charge', d3.forceManyBody().strength(-500))
    .force('center', d3.forceCenter(width/2, height/2))
    .force('x', d3.forceX(width/2))
    .force('y', d3.forceY((d) => {
        let padding = height/(nodes.length+1);
        let center = height - 2*padding;
        let unit = center/max_difficulty;
        return padding + unit*d.difficulty;
    }))
    .on('tick', tick);

let link_v = svg.append('svg:g').selectAll('link_v');
let node_v = svg.append('svg:g').selectAll('node_v');

function tick() {
    node_v.attr('transform', (d) => `translate(${d.x},${d.y})`);

    link_v
        .attr('x1', (d) => d.source.x).attr('y1', (d) => d.source.y)
        .attr('x2', (d) => d.target.x).attr('y2', (d) => d.target.y);
}

function processData() {
    // only first proof is considered
    // theorems need to be in topological order
    // TODO add topological sort
    nodes = [];
    links = [];
}

function restart() {
    link_v = link_v.data(links);
    link_v = link_v.enter().append('line')
        .attr('x1', (d) => d.source.x).attr('y1', (d) => d.source.y)
        .attr('x2', (d) => d.target.x).attr('y2', (d) => d.target.y)
        .attr('class', 'link')
        .merge(link_v);

    link_v.exit().remove();

    node_v = node_v.data(nodes);
    node_v = node_v.enter().append('svg:g').append('svg:node_v')
        .attr('class', 'node')
        .attr('r', 12)
        .style('fill', (d) => colors(d.id))
        .style('stroke', (d) => d3.rgb(colors(d.id)).darker().toString());
        .merge(node_v);

    node_v.exit().remove();

    force.alphaTarget(0.3).restart();
}

function import_graph(text) {
    // TODO
    data = JSON.parse(text);
    processData();
    restart();
}
function export_graph() {
    console.log(JSON.stringify(data));
}

processData();
restart();
</script>