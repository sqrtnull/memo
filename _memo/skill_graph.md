---
layout: post
title: Math Skill Graph
author: sqrtnull
date: 2020-10-23
category: d3
---


```
Data:

Axiom = {
  id, statement
}
Theorem = {
  id, statement, proofs = [Proof]
}
Proof = {
  id, statement, assumptions = [Axiom or Theorem]
}
// possibly add something like restrictions to group similar theorems

Equations:

difficulty = {
  1 if Axiom
  max(difficulty(assumptions))+size(assumptions) if Proof
  min(difficulty(proofs)) if Theorem
}

Modes:

Proof Graph = {
  creates a directed graph based on proofs / assumptions -> theorem
}

Equivalence Graph = {
  creates a directed graph based on what theorems are equivalent and the strength of them
}

TODO:

add links(edges) based on difficulty difference and change forceY based on difficulty to balance the graph ( idk if links are really necessary )
make an UI for adding Theorems, Proofs, and Axioms
change color based on difficulty
```

<div id='graph'></div>

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
.link {
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

let nodes = [
];
let links = [
];

let data = {
    Axiom: [
    { id: 0, statement: 'a', link: '' },
    { id: 1, statement: 'b', link: '' }
    ],
    Theorem: [
    { id: 2, statement: 'c', link: '', proofs: [
        { statement: 'e', link: '', assumptions: [0] }
    ] },
    { id: 3, statement: 'd', link: '', proofs: [
        { statement: 'f', link: '', assumptions: [0,1] }
    ] }
    ]
};

let max_difficulty=1;

const epsilon = 0.0001;

const force = d3.forceSimulation()
    .force('link', d3.forceLink().id((d) => d.id).distance(100).strength(epsilon))
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


// svg for edge
svg.append('svg:defs').append('svg:marker').attr('id', 'end-arrow')
    .attr('viewBox', '0 -5 10 10')
    .attr('refX', 6)
    .attr('markerWidth', 3)
    .attr('markerHeight', 3)
    .attr('orient', 'auto')
    .append('svg:path')
    .attr('d', 'M0,-5L10,0L0,5')
    .attr('fill', '#000');
svg.append('svg:defs').append('svg:marker').attr('id', 'start-arrow')
    .attr('viewBox', '0 -5 10 10')
    .attr('refX', 4)
    .attr('markerWidth', 3)
    .attr('markerHeight', 3)
    .attr('orient', 'auto')
    .append('svg:path')
    .attr('d', 'M10,-5L0,0L10,5')
    .attr('fill', '#000');

let path = svg.append('svg:g').selectAll('link');
let circle = svg.append('svg:g').selectAll('g');

function tick() {
    path.attr('x1', (d) => d.source.x)
        .attr('y1', (d) => d.source.y)
        .attr('x2', (d) => d.target.x)
        .attr('y2', (d) => d.target.y);

    circle.attr('transform', (d) => `translate(${d.x},${d.y})`);
}

function processData() {
    // only first proof is considered
    // theorems need to be in topological order
    // TODO add topological sort
    nodes = [];
    let map = new Map();
    data.Axiom.forEach( (e) => {
        map.set(e.id, {
            id: e.id,
            statement: e.statement,
            link: e.link,
            isAxiom: true,
            proofs: [],
            difficulty: 1
        });
    });
    data.Theorem.forEach( (e) => {
        let diff=-1;
        if(e.proofs.length>0) {
            e.proofs[0].assumptions.forEach( (ap) => {
                if(diff<map.get(ap).difficulty) {
                    diff=map.get(ap).difficulty;
                }
                links.push({ source: map.get(ap).id, target: e.id});
            });
            diff+=e.proofs[0].assumptions.length;
        }
        map.set(e.id,{
            id: e.id,
            statement: e.statement,
            link: e.link,
            isAxiom: false,
            proofs: e.proofs,
            difficulty: diff
        });
        if(max_difficulty<diff) max_difficulty=diff;
    });
    for(let [key, value] of map) {
        nodes.push(value);
    }

    // assume nodes are sorted by id
    // bisect = d3.bisector((d) => d.id);
    // for(link of links) {
    //     link.source = bisect.left(nodes,link.source);
    //     link.target = bisect.left(nodes,link.target);
    // }
}

function restart() {
    path = path.data(links, (d) => d.id);

  // add new links
    path = path.enter().append('line')
        .attr('x1', (d) => d.source.x)
        .attr('y1', (d) => d.source.y)
        .attr('x2', (d) => d.target.x)
        .attr('y2', (d) => d.target.y)
        .attr('class', 'link')
        .merge(path)

    path.exit().remove();

    circle = circle.data(nodes, (d) => d.id);
    circle.exit().remove();
    const g = circle.enter().append('svg:g');
    g.append('svg:circle')
        .attr('class', 'node')
        .attr('r', 12)
        .style('fill', (d) => colors(d.id))
        .style('stroke', (d) => d3.rgb(colors(d.id)).darker().toString());

    circle = g.merge(circle);
    force.nodes(nodes).force('link').links(links);
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