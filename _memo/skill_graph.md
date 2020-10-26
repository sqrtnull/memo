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
{id:0},
{id:1},
{id:2}
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
    ],
};

const force = d3.forceSimulation()
    .force('link', d3.forceLink().id((d) => d.id).distance(150))
    .force('charge', d3.forceManyBody().strength(-500))
    .force('center', d3.forceCenter(width/2, height/2))
    .force('x', d3.forceX(width/2))
    .force('y', d3.forceY(height/2))
    .on('tick', tick);

let circle = svg.append('svg:g').selectAll('g');

function tick() {
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
            })
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
    });
    console.log(map);
    for(let [key, value] of map) {
        nodes.push(value);
    }
    nodes.forEach( e => console.log(e) );
}

function restart() {
    circle = circle.data(nodes, (d) => d.id);
    circle.exit().remove();
    const g = circle.enter().append('svg:g');
    g.append('svg:circle')
        .attr('class', 'node')
        .attr('r', 12)
        .style('fill', (d) => colors(d.id))
        .style('stroke', (d) => d3.rgb(colors(d.id)).darker().toString());

    circle = g.merge(circle);
    force.nodes(nodes);
    force.alphaTarget(0.3).restart();
}

processData();
restart();
</script>