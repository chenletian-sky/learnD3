<template>
</template>
<script setup lang="ts">
import * as d3 from 'd3'
import {onMounted} from "vue";
import {MarginType,ConnectedScatterplotDataType} from "../interface";

import ConnectedScatterplotData from '../../assets/json/ConnectedScatterplotData.json'
import TemporalforcedirectedgraphData from '../../assets/json/TemporalforcedirectedgraphData.json'
import SmoothZoomingData from '../../assets/json/SmoothZoomingData.json'
import CollapsibleTreeData from '../../assets/json/CollapsibleTreeData.json'

const connectedScatterplotData:ConnectedScatterplotDataType[] = ConnectedScatterplotData
const length = (path:any) => (d3.create("svg:path")!.attr("d", path)!.node()! as any).getTotalLength()
const d3DrawConnectedScatterplot = (data:ConnectedScatterplotDataType[]) => {
  // Declare the chart dimensions and margins.
  const width = 928;
  const height = 720;
  const margin:MarginType = {top: 20, right: 30, bottom: 30, left: 40}

  // Declare the positional encodings.
  const x = d3.scaleLinear()
      .domain(d3.extent(data, (d:any) => d.miles)).nice()
      .range([margin.left, width -margin.right]);

  const y = d3.scaleLinear()
      .domain(d3.extent(data, (d:any) => d.gas)).nice()
      .range([height - margin.bottom, margin.top]);

  const line = d3.line()
      .curve(d3.curveCatmullRom)
      .x((d:any) => x(d.miles))
      .y((d:any) => y(d.gas))

  const svg = d3.select('#d3-root').append('svg')
      .attr("width", width)
      .attr("height", height)
      .attr("viewBox", [0, 0, width, height])
      .attr("style", "max-width: 100%; height: auto;");

  const l = length(line(data));

  svg.append("g")
      .attr("transform", `translate(0,${height - margin.bottom})`)
      .call(d3.axisBottom(x).ticks(width / 80))
      .call(g => g.select(".domain").remove())
      .call(g => g.selectAll(".tick line").clone()
          .attr("y2", -height)
          .attr("stroke-opacity", 0.1))
      .call(g => g.append("text")
          .attr("x", width - 4)
          .attr("y", -4)
          .attr("font-weight", "bold")
          .attr("text-anchor", "end")
          .attr("fill", "currentColor")
          .text("Miles per person per year"));

  svg.append("g")
      .attr("transform", `translate(${margin.left},0)`)
      .call(d3.axisLeft(y).ticks(null, "$.2f"))
      .call(g => g.select(".domain").remove())
      .call(g => g.selectAll(".tick line").clone()
          .attr("x2", width)
          .attr("stroke-opacity", 0.1))
      .call(g => g.select(".tick:last-of-type text").clone()
          .attr("x", 4)
          .attr("text-anchor", "start")
          .attr("font-weight", "bold")
          .text("Cost per gallon"));

  svg.append("path")
      .datum(data)
      .attr("fill", "none")
      .attr("stroke", "black")
      .attr("stroke-width", 2.5)
      .attr("stroke-linejoin", "round")
      .attr("stroke-linecap", "round")
      .attr("stroke-dasharray", `0,${l}`)
      .attr("d", line)
      .transition()
      .duration(5000)
      .ease(d3.easeLinear)
      .attr("stroke-dasharray", `${l},${l}`);

  svg.append("g")
      .attr("fill", "white")
      .attr("stroke", "black")
      .attr("stroke-width", 2)
      .selectAll("circle")
      .data(data)
      .join("circle")
      .attr("cx", (d:any) => x(d.miles))
      .attr("cy", (d:any) => y(d.gas))
      .attr("r", 3);

  const label = svg.append("g")
      .attr("font-family", "sans-serif")
      .attr("font-size", 10)
      .selectAll()
      .data(data)
      .join("text")
      .attr("transform", (d:any) => `translate(${x(d.miles)},${y(d.gas)})`)
      .attr("fill-opacity", 0)
      .text((d:any) => d.year)
      .attr("stroke", "white")
      .attr("paint-order", "stroke")
      .attr("fill", "currentColor")
      .each(function(d) {
        const t = d3.select(this);
        switch (d.side) {
          case "top": t.attr("text-anchor", "middle").attr("dy", "-0.7em"); break;
          case "right": t.attr("dx", "0.5em").attr("dy", "0.32em").attr("text-anchor", "start"); break;
          case "bottom": t.attr("text-anchor", "middle").attr("dy", "1.4em"); break;
          case "left": t.attr("dx", "-0.5em").attr("dy", "0.32em").attr("text-anchor", "end"); break;
        }
      });

  label.transition()
      .delay((_, i) => length(line(data.slice(0, i + 1))) / l * (5000 - 125))
      .attr("fill-opacity", 1);

  return svg.node();
}
const temporalforcedirectedgraphData = TemporalforcedirectedgraphData
const temporalforcedirectedgraphDataConvertData =() => {
  let nodes = temporalforcedirectedgraphData.nodes
  let links = temporalforcedirectedgraphData.links
  nodes.forEach((d:any) => {
    d.start = d3.isoParse(d.start);
    d.end = d3.isoParse(d.end);
  })
  links.forEach((d:any) => {
    d.start = d3.isoParse(d.start);
    d.end = d3.isoParse(d.end);
  })

  return {nodes, links}
}
const contains = ({start, end}, time) => start <= time && time < end
console.log('temporalforcedirectedgraphDataConvertData',temporalforcedirectedgraphDataConvertData())
const times = d3.scaleTime()
    .domain([d3.min(temporalforcedirectedgraphDataConvertData().nodes, (d:any) => d.start), d3.max(temporalforcedirectedgraphDataConvertData().nodes, d => d.end)])
    .ticks(1000)
    .filter(time => temporalforcedirectedgraphDataConvertData().nodes.some(d => contains(d, time)))
const drag = (simulation:any) => {

  function dragstarted(event, d) {
    if (!event.active) simulation.alphaTarget(0.3).restart();
    d.fx = d.x;
    d.fy = d.y;
  }

  function dragged(event, d) {
    d.fx = event.x;
    d.fy = event.y;
  }

  function dragended(event, d) {
    if (!event.active) simulation.alphaTarget(0);
    d.fx = null;
    d.fy = null;
  }

  return d3.drag()
      .on("start", dragstarted)
      .on("drag", dragged)
      .on("end", dragended);
}
const d3DrawTemporalForceDirectedGraph = () => {
  const width = 928;
  const height = 680;

  const simulation = d3.forceSimulation()
      .force("charge", d3.forceManyBody())
      .force("link", d3.forceLink().id((d:any) => d.id))
      .force("x", d3.forceX())
      .force("y", d3.forceY())
      .on("tick", ticked);

  const svg = d3.select('#d3-root')
      .append('svg')
      .attr("viewBox", [-width / 2, -height / 2, width, height])
      .attr("width", width)
      .attr("height", height)
      .attr("style", "max-width: 100%; height: auto;");

  let link = svg.append("g")
      .attr("stroke", "#999")
      .attr("stroke-opacity", 0.6)
      .selectAll("line");

  let node = svg.append("g")
      .attr("stroke", "#fff")
      .attr("stroke-width", 1.5)
      .selectAll("circle");

  function ticked() {
    node.attr("cx", (d:any) => d.x)
        .attr("cy", (d:any) => d.y);

    link.attr("x1", (d:any) => d.source.x)
        .attr("y1", (d:any) => d.source.y)
        .attr("x2", (d:any) => d.target.x)
        .attr("y2", (d:any) => d.target.y);
  }

  // invalidation.then(() => simulation.stop());

  return Object.assign(svg.node(), {
    update({nodes, links}) {

      // Make a shallow copy to protect against mutation, while
      // recycling old nodes to preserve position and velocity.
      const old = new Map(node.data().map(d => [d.id, d]));
      nodes = nodes.map(d => ({...old.get(d.id), ... d}));
      links = links.map(d => ({...d}));

      node = node
          .data(nodes, (d:any) => d.id)
          .join(enter => enter.append("circle")
              .attr("r", 5)
              .call(drag(simulation))
              .call(node => node.append("title").text((d:any) => d.id)));

      link = link
          .data(links, d => [d.source, d.target])
          .join("line");

      simulation.nodes(nodes);
      simulation.force("link").links(links);
      simulation.alpha(1).restart().tick();
      ticked(); // render now!
    }
  });
}
const update =() => {
  const nodes = temporalforcedirectedgraphDataConvertData().nodes.filter(d => contains(d, '2009-06-04T09:01:40.000Z'));
  const links = temporalforcedirectedgraphDataConvertData().links.filter(d => contains(d, '2009-06-04T09:01:40.000Z'));
  d3DrawTemporalForceDirectedGraph().update({nodes, links});
}
const xz = [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57]
const yz = [[0.18852117600542959,0.17764283655038482,0.10514968554042081,0.1663986733924981,0.18363751457286964,0.16061744415086676,0.18894388893729952,0.15647534696200996,0.16301887277000335,0.15007247686300337,0.18248206840574918,0.2863691039192978,0.37555308250062835,0.7346998401278115,1.194038839950954,1.6169211967430743,2.0104140331373084,2.2263308288235026,2.0234119005662876,1.5747794080072959,1.0932805105689105,0.6481278563224457,0.39727523177074314,0.24738322059769277,0.16413907194425828,0.20742844888531722,0.20068309946822804,0.19337623096724818,0.14231504341413975,0.11073570642938373,0.1717375749931727,0.20048324700587045,0.2169329393320908,0.3987509965633793,0.5369296795996219,0.6820836566931705,0.9748035189759328,1.266781137216186,1.5211646903526181,1.739886750675861,1.71816384737175,1.705084117627714,1.4597248590462788,1.2011408272899824,0.8690411923155263,0.5964346767165839,0.39308648729324835,0.314394435585726,0.2366656589118915,0.19562039293301847,0.1416850589234462,0.1863822362916922,0.1992223261138116,0.14417483132047476,0.15998728056232428,0.14982690725833775,0.173339451561249,0.17446719980307956],[0.12628392789867343,0.13846018475975316,0.13199641287670486,0.10014494555639895,0.14424919698013772,0.15825407291278726,0.17546959881788524,0.15438985172047623,0.164079608426539,0.1564112308220343,0.21372704345595486,0.1618265704594668,0.3311765135246219,0.4608178884061737,0.8091962662557954,1.1834346057881409,1.6092688144734406,1.8166314607929683,1.7327661145826825,1.4471700990045813,1.0890316725404594,0.6933954409059309,0.4116521822406543,0.2880762531027094,0.26829743756309943,0.26656251261537206,0.32079443793383156,0.45003266746952986,0.6201645725083993,0.7671713792597543,1.1166800569378779,1.310103808063756,1.650402291148739,1.733308645757784,1.8718084544313616,1.8178728419735446,1.965307565827475,2.779890182173232,2.6907785349543167,1.3066819902985172,0.5987882957624084,0.4540878882974566,0.4108507321840361,0.7469925977451438,1.4651245227597336,1.5801765409313724,1.0232873040222579,0.434070799426692,0.21132820873625324,0.18677668067512906,0.19081239884499393,0.14836134570135306,0.11176378075923474,0.17243154400034869,0.18095705012947583,0.1799853099413306,0.19167781654163374,0.1738086095376299],[4.0105593613514925,3.4548071689236908,2.565177913998043,1.7501486699515183,1.0690655043171584,0.6629583640893424,0.3714163811042912,0.2040417132206067,0.19918105391128732,0.20482941913869168,0.18684073894774006,0.30763549512130767,0.4254638251929521,0.53728024291924,0.8536380642953859,1.2080507208651539,1.713884465694017,2.233886926667315,2.682971858538561,3.0965333981319785,3.33535025295756,3.3872187443800286,3.2676458101381,2.9465679976888866,2.4579443410136266,1.965078819172184,1.502085778673884,1.0478848848357492,0.764845181149424,0.4786803017186111,0.4052873733930237,0.2971549212074363,0.20358225220629217,0.21249574333979546,0.2010892928944042,0.1758144828003462,0.14073134235256943,0.1226183840173262,0.14711449634762047,0.1668586360882815,0.15396232985449063,0.19648867776703044,0.19618199227320246,0.1792391782429488,0.1260857220585911,0.11587100241990299,0.13028097541139144,0.11142504791935592,0.1949409296922082,0.11163104710571255,0.12251755497388159,0.17108134603795702,0.13939370635223927,0.16951460225146608,0.11787378154295246,0.15860141453353196,0.12675626939387086,0.12099874413120276],[0.29119131444591123,0.4044805105573802,0.5706672567045956,0.87877532427803,1.2649944965711186,1.7828211224085626,2.3684779037975843,3.152581588759602,3.8339547507456495,4.402929099224639,4.7581595344288194,4.921717320046006,4.76835371182922,4.302644767331838,3.621148058578596,2.957611199152744,2.2654393130750727,1.6095270596528017,1.1700795069880854,0.7996072146509939,0.49269371451815674,0.3238261055359718,0.26390949509165845,0.2024061435821548,0.21303559446713824,0.13137694105877556,0.12680622442830372,0.11204543345563683,0.11864224849280515,0.16319020981570742,0.1746456369161852,0.10697726504455307,0.18953345389366824,0.16112732698426274,0.12400386660523677,0.1363546063644361,0.11713147987852887,0.1400722242568826,0.1562309825928258,0.19334948411697117,0.20497302615886212,0.3288516974158923,0.32971717853787547,0.4556924116105323,0.6069312894398441,0.7812201739786824,0.8545721122064965,1.0585383233512415,1.120051987527269,1.1312982035334964,1.1608617212394345,1.1059797884052773,0.9891988487786296,0.8504896245655432,0.93868831632906,2.0745283034569315,1.0986454810987913,0.35987033196177254],[4.253799468062936,3.833024551138105,2.99600612127205,2.0632481645194596,1.1732025991918753,0.6205824795168979,0.3322710586180084,0.21386554411820546,0.1339673082900486,0.1786043975005656,0.19314466080436016,0.18996307358712616,0.1110131362197562,0.12387815195925349,0.14217833181430448,0.19669605139285437,0.10480894705251882,0.1788646603283421,0.10188822455816736,0.10086939106876003,0.16761210765598725,0.15301338623267755,0.1832774519717074,0.12224933404706068,0.10274068018219401,0.13279144074829086,0.16158834441388376,0.15127627685514025,0.18825200472255466,0.1866218179189608,0.17560380686318797,0.15607968065226296,0.17909531017650482,0.18779576593850872,0.1714915349029879,0.15542002740291092,0.10307617477956905,0.18098566996163956,0.14709200635026687,0.10236499574518824,0.11013158933456027,0.13106593329203037,0.17143662126475848,0.18504381409263507,0.1514662891186713,0.2286350560462567,0.24047538618235137,0.39250504204003517,0.5830950826973955,0.8643079111193427,1.0909020986668116,1.1360560927914136,0.9478522552766947,0.6788875468130939,0.4272543510198053,0.276771311542786,0.25011798194392043,0.6293131792003375]]
const n = 5
const m = 58
// Returns an array of m pseudorandom, smoothly-varying non-negative numbers.
// Inspired by Lee Byron’s test data generator.
// http://leebyron.com/streamgraph/
function bumps(m) {
  const values = [];

  // Initialize with uniform random values in [0.1, 0.2).
  for (let i = 0; i < m; ++i) {
    values[i] = 0.1 + 0.1 * Math.random();
  }

  // Add five random bumps.
  for (let j = 0; j < 5; ++j) {
    const x = 1 / (0.1 + Math.random());
    const y = 2 * Math.random() - 0.5;
    const z = 10 / (0.1 + Math.random());
    for (let i = 0; i < m; i++) {
      const w = (i / m - y) * z;
      values[i] += x * Math.exp(-w * w);
    }
  }

  // Ensure all values are positive.
  for (let i = 0; i < m; ++i) {
    values[i] = Math.max(0, values[i]);
  }

  return values;
}
const d3DrawStackedToGroupedBars = () => {
  const width = 928;
  const height = 500;
  const marginTop = 0;
  const marginRight = 0;
  const marginBottom = 10;
  const marginLeft = 0;

  const y01z = d3.stack()
      .keys(d3.range(n))
      (d3.transpose(yz)) // stacked yz
      .map((data, i) => data.map(([y0, y1]) => [y0, y1, i]));

  const yMax = d3.max(yz, y => d3.max(y));
  const y1Max = d3.max(y01z, y => d3.max(y, d => d[1]));

  const x = d3.scaleBand()
      .domain(xz)
      .rangeRound([marginLeft, width - marginRight])
      .padding(0.08);

  const y = d3.scaleLinear()
      .domain([0, y1Max])
      .range([height - marginBottom, marginTop]);

  const color = d3.scaleSequential(d3.interpolateBlues)
      .domain([-0.5 * n, 1.5 * n]);

  const svg = d3.select('#d3-root')
      .append('svg')
      .attr("viewBox", [0, 0, width, height])
      .attr("width", width)
      .attr("height", height)
      .attr("style", "max-width: 100%; height: auto;");

  const rect = svg.selectAll("g")
      .data(y01z)
      .join("g")
      .attr("fill", (d, i) => color(i))
      .selectAll("rect")
      .data(d => d)
      .join("rect")
      .attr("x", (d, i) => x(i))
      .attr("y", height - marginBottom)
      .attr("width", x.bandwidth())
      .attr("height", 0);

  svg.append("g")
      .attr("transform", `translate(0,${height - marginBottom})`)
      .call(d3.axisBottom(x).tickSizeOuter(0).tickFormat(() => ""));

  function transitionGrouped() {
    y.domain([0, yMax]);

    rect.transition()
        .duration(500)
        .delay((d, i) => i * 20)
        .attr("x", (d, i) => x(i) + x.bandwidth() / n * d[2])
        .attr("width", x.bandwidth() / n)
        .transition()
        .attr("y", d => y(d[1] - d[0]))
        .attr("height", d => y(0) - y(d[1] - d[0]));
  }

  function transitionStacked() {
    y.domain([0, y1Max]);

    rect.transition()
        .duration(500)
        .delay((d, i) => i * 20)
        .attr("y", d => y(d[1]))
        .attr("height", d => y(d[0]) - y(d[1]))
        .transition()
        .attr("x", (d, i) => x(i))
        .attr("width", x.bandwidth());
  }

  function update(layout:string) {
    if (layout === "stacked") transitionStacked();
    else transitionGrouped();
  }

  return Object.assign(svg.node(), {update});
}
const smoothZoomingData = SmoothZoomingData
const theta = Math.PI * (3 - Math.sqrt(5))
const d3DrawSmoothZooming = (data:any) => {
  const height = 500
  const width = 500
  const radius = 6
  let step = radius * 2
  let currentTransform = [width / 2, height / 2, height];

  const svg = d3.select('#d3-root').append('svg')
      .attr("viewBox", [0, 0, width, height])

  const g = svg.append("g");

  g.selectAll("circle")
      .data(data)
      .join("circle")
      .attr("cx", ([x]) => x)
      .attr("cy", ([, y]) => y)
      .attr("r", radius)
      .attr("fill", (d, i) => d3.interpolateRainbow(i / 360))

  function transition() {
    const d = data[Math.floor(Math.random() * data.length)];
    const i = d3.interpolateZoom(currentTransform, [...d, radius * 2 + 1]);

    g.transition()
        .delay(250)
        .duration(i.duration)
        .attrTween("transform", () => t => transform(currentTransform = i(t)))
        .on("end", transition);
  }

  function transform([x, y, r]) {
    return `
      translate(${width / 2}, ${height / 2})
      scale(${height / r})
      translate(${-x}, ${-y})
    `;
  }

  return svg.call(transition).node();
}
const collapsibleTreeData = CollapsibleTreeData
const d3DrawCollapsibleTree = (data:any) => {
  // Specify the charts’ dimensions. The height is variable, depending on the layout.
  const width = 928;
  const marginTop = 10;
  const marginRight = 10;
  const marginBottom = 10;
  const marginLeft = 40;

  // Rows are separated by dx pixels, columns by dy pixels. These names can be counter-intuitive
  // (dx is a height, and dy a width). This because the tree must be viewed with the root at the
  // “bottom”, in the data domain. The width of a column is based on the tree’s height.
  const root = d3.hierarchy(data);
  const dx = 10;
  const dy = (width - marginRight - marginLeft) / (1 + root.height);

  // Define the tree layout and the shape for links.
  const tree = d3.tree().nodeSize([dx, dy]);
  const diagonal = d3.linkHorizontal().x(d => d.y).y(d => d.x);

  // Create the SVG container, a layer for the links and a layer for the nodes.
  const svg = d3.select('#d3-root').append('svg')
      .attr("width", width)
      .attr("height", dx)
      .attr("viewBox", [-marginLeft, -marginTop, width, dx])
      .attr("style", "max-width: 100%; height: auto; font: 10px sans-serif; user-select: none;");

  const gLink = svg.append("g")
      .attr("fill", "none")
      .attr("stroke", "#555")
      .attr("stroke-opacity", 0.4)
      .attr("stroke-width", 1.5);

  const gNode = svg.append("g")
      .attr("cursor", "pointer")
      .attr("pointer-events", "all");

  function update(event, source) {
    const duration = event?.altKey ? 2500 : 250; // hold the alt key to slow down the transition
    const nodes = root.descendants().reverse();
    const links = root.links();

    // Compute the new tree layout.
    tree(root);

    let left = root;
    let right = root;
    root.eachBefore(node => {
      if (node.x < left.x) left = node;
      if (node.x > right.x) right = node;
    });

    const height = right.x - left.x + marginTop + marginBottom;

    const transition = svg.transition()
        .duration(duration)
        .attr("height", height)
        .attr("viewBox", [-marginLeft, left.x - marginTop, width, height])
        .tween("resize", window.ResizeObserver ? null : () => () => svg.dispatch("toggle"));

    // Update the nodes…
    const node = gNode.selectAll("g")
        .data(nodes, d => d.id);

    // Enter any new nodes at the parent's previous position.
    const nodeEnter = node.enter().append("g")
        .attr("transform", d => `translate(${source.y0},${source.x0})`)
        .attr("fill-opacity", 0)
        .attr("stroke-opacity", 0)
        .on("click", (event, d) => {
          d.children = d.children ? null : d._children;
          update(event, d);
        });

    nodeEnter.append("circle")
        .attr("r", 2.5)
        .attr("fill", d => d._children ? "#555" : "#999")
        .attr("stroke-width", 10);

    nodeEnter.append("text")
        .attr("dy", "0.31em")
        .attr("x", d => d._children ? -6 : 6)
        .attr("text-anchor", d => d._children ? "end" : "start")
        .text(d => d.data.name)
        .attr("stroke-linejoin", "round")
        .attr("stroke-width", 3)
        .attr("stroke", "white")
        .attr("paint-order", "stroke");

    // Transition nodes to their new position.
    const nodeUpdate = node.merge(nodeEnter).transition(transition)
        .attr("transform", d => `translate(${d.y},${d.x})`)
        .attr("fill-opacity", 1)
        .attr("stroke-opacity", 1);

    // Transition exiting nodes to the parent's new position.
    const nodeExit = node.exit().transition(transition).remove()
        .attr("transform", d => `translate(${source.y},${source.x})`)
        .attr("fill-opacity", 0)
        .attr("stroke-opacity", 0);

    // Update the links…
    const link = gLink.selectAll("path")
        .data(links, d => d.target.id);

    // Enter any new links at the parent's previous position.
    const linkEnter = link.enter().append("path")
        .attr("d", d => {
          const o = {x: source.x0, y: source.y0};
          return diagonal({source: o, target: o});
        });

    // Transition links to their new position.
    link.merge(linkEnter).transition(transition)
        .attr("d", diagonal);

    // Transition exiting nodes to the parent's new position.
    link.exit().transition(transition).remove()
        .attr("d", d => {
          const o = {x: source.x, y: source.y};
          return diagonal({source: o, target: o});
        });

    // Stash the old positions for transition.
    root.eachBefore(d => {
      d.x0 = d.x;
      d.y0 = d.y;
    });
  }

  // Do the first update to the initial configuration of the tree — where a number of nodes
  // are open (arbitrarily selected as the root, plus nodes with 7 letters).
  root.x0 = dy / 2;
  root.y0 = 0;
  root.descendants().forEach((d, i) => {
    d.id = i;
    d._children = d.children;
    if (d.depth && d.data.name.length !== 7) d.children = null;
  });

  update(null, root);

  return svg.node();
}
const d3DrawArcTween = () => {
  // Compute the dimensions; the width is provided by Observable.
  const width = 500
  const height = Math.min(500, width / 2);
  const outerRadius = height / 2 - 10;
  const innerRadius = outerRadius * 0.75;

  // https://tauday.com/tau-manifesto
  const tau = 2 * Math.PI;

  // Create the SVG container, and apply a transform such that the origin is the
  // center of the canvas. This way, we don’t need to position arcs individually.
  const svg = d3.select('#d3-root').append('svg')
      .attr("viewBox", [0, 0, width, height]);
  const g = svg.append("g").attr("transform", "translate(" + width / 2 + "," + height / 2 + ")");

  // An arc function with all values bound except the endAngle. So, to compute an
  // SVG path string for a given angle, we pass an object with an endAngle
  // property to the arc function, and it will return the corresponding string.
  const arc = d3.arc()
      .innerRadius(innerRadius)
      .outerRadius(outerRadius)
      .startAngle(0);

  // Add the background arc, from 0 to 100% (tau).
  const background = g.append("path")
      .datum({endAngle: tau})
      .style("fill", "#ddd")
      .attr("d", arc);

  // Add the foreground arc in orange, currently showing 12.7%.
  const foreground = g.append("path")
      .datum({endAngle: 0.127 * tau})
      .style("fill", "orange")
      .attr("d", arc);

  // Every so often, start a transition to a new random angle. The attrTween
  // definition is encapsulated in a separate function (a closure) below.
  const interval = d3.interval(function() {
    foreground.transition()
        .duration(750)
        .attrTween("d", arcTween(Math.random() * tau));
  }, 1500);

  // Stop the interval when this block of code updates
  invalidation.then(() => interval.stop());

  // Returns a tween for a transition’s "d" attribute, transitioning any selected
  // arcs from their current angle to the specified new angle.
  function arcTween(newAngle) {

    // The function passed to attrTween is invoked for each selected element when
    // the transition starts, and for each element returns the interpolator to use
    // over the course of transition. This function is thus responsible for
    // determining the starting angle of the transition (which is pulled from the
    // element’s bound datum, d.endAngle), and the ending angle (simply the
    // newAngle argument to the enclosing function).
    // https://d3js.org/d3-transition/modifying#transition_attrTween
    return function(d) {

      // To interpolate between the two angles, we use the default d3.interpolate.
      // (Internally, this maps to d3.interpolateNumber, since both of the
      // arguments to d3.interpolate are numbers.) The returned function takes a
      // single argument t and returns a number between the starting angle and the
      // ending angle. When t = 0, it returns d.endAngle; when t = 1, it returns
      // newAngle; and for 0 < t < 1 it returns an angle in-between.
      const interpolate = d3.interpolate(d.endAngle, newAngle);

      // The return value of the attrTween is also a function: the function that
      // we want to run for each tick of the transition. Because we used
      // attrTween("d"), the return value of this last function will be set to the
      // "d" attribute at every tick. (It’s also possible to use transition.tween
      // to run arbitrary code for every tick, say if you want to set multiple
      // attributes from a single function.) The argument t ranges from 0, at the
      // start of the transition, to 1, at the end.
      return function(t) {

        // Calculate the current arc angle based on the transition time, t. Since
        // the t for the transition and the t for the interpolate both range from
        // 0 to 1, we can pass t directly to the interpolator.
        //
        // Note that the interpolated angle is written into the element’s bound
        // data object! This is important: it means that if the transition were
        // interrupted, the data bound to the element would still be consistent
        // with its appearance. Whenever we start a new arc transition, the
        // correct starting angle can be inferred from the data.
        d.endAngle = interpolate(t);

        // Lastly, compute the arc path given the updated data! In effect, this
        // transition uses data-space interpolation: the data is interpolated
        // (that is, the end angle) rather than the path string itself.
        // Interpolating the angles in polar coordinates, rather than the raw path
        // string, produces valid intermediate arcs during the transition.
        return arc(d);
      };
    };
  }

  // Return the svg node to be displayed.
  return svg.node();

}

onMounted(() => {
  d3DrawConnectedScatterplot(connectedScatterplotData)
  // d3DrawStackedToGroupedBars().update('stacked')
  d3DrawStackedToGroupedBars().update('grouped')
  // d3DrawSmoothZooming(smoothZoomingData)
  d3DrawCollapsibleTree(collapsibleTreeData)
  d3DrawArcTween()
})
</script>

<style scoped>

</style>