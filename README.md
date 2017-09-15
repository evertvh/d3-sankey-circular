# d3-sankey-circular

A fork of the d3-sankey library (https://github.com/d3/d3-sankey) to allow circular links (ie cyclic graphs).

The library contains a portion of code from Colin Fergus' bl.ock https://gist.github.com/cfergus/3956043 to detect circular links. 

## Usage

var sankey = d3.sankey();

## API Reference

<b>*The API follows the original d3-sankey library, with additional options to allow the circular links to be laid out and drawn.*</b>

<a href="#sankey" name="sankey">#</a> d3.<b>sankey</b>() [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L52 "Source")

Constructs a new Sankey generator with the default settings.

<a href="#_sankey" name="_sankey">#</a> <i>sankey</i>(<i>arguments</i>…) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L62 "Source")

Computes the node and link positions for the given *arguments*, returning a *graph* representing the Sankey layout. The returned *graph* has the following properties:

* *graph*.nodes - the array of [nodes](#sankey_nodes)
* *graph*.links - the array of [links](#sankey_links)

<a name="sankey_nodes" href="#sankey_nodes">#</a> <i>sankey</i>.<b>nodes</b>([<i>nodes</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L93 "Source")

If *nodes* is specified, sets the Sankey generator’s nodes accessor to the specified function or array and returns this Sankey generator. If *nodes* is not specified, returns the current nodes accessor, which defaults to:

```js
function nodes(graph) {
  return graph.nodes;
}
```

If *nodes* is specified as a function, the function is invoked when the Sankey layout is [generated](#_sankey), being passed any arguments passed to the Sankey generator. This function must return an array of nodes. If *nodes* is not a function, it must be a constant array of *nodes*.

Each *node* must be an object. The following properties are assigned by the [Sankey generator](#_sankey):

* *node*.sourceLinks - the array of outgoing [links](#sankey_links) which have this node as their source
* *node*.targetLinks - the array of incoming [links](#sankey_links) which have this node as their target
* *node*.value - the node’s value; the sum of *link*.value for the node’s incoming [links](#sankey_links)
* *node*.index - the node’s zero-based index within the array of nodes
* *node*.depth - the node’s zero-based graph depth, derived from the graph topology
* *node*.column - the node’s zero-based depth, as drawn, going from left to right. Column is derived from *node*.depth and the [*sankey*.nodeAlign](#sankey_nodeAlign) setting.
* *node*.height - the node’s zero-based graph height, derived from the graph topology
* *node*.x0 - the node’s minimum horizontal position, derived from *node*.column
* *node*.x1 - the node’s maximum horizontal position (*node*.x0 + [*sankey*.nodeWidth](#sankey_nodeWidth))
* *node*.y0 - the node’s minimum vertical position
* *node*.y1 - the node’s maximum vertical position (*node*.y1 - *node*.y0 is proportional to *node*.value)
* *node*.partOfCycle - set to *true* if the node has incoming or outgoing links that are circular
* *node*.circularLinkType - set to either *top* or *bottom*, if *node.partOfCycle* is *true*, which relates to whether circular links are drawn above or below the main part of the graph

See also [*sankey*.links](#sankey_links).

<a name="sankey_links" href="#sankey_links">#</a> <i>sankey</i>.<b>links</b>([<i>links</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L97 "Source")

If *links* is specified, sets the Sankey generator’s links accessor to the specified function or array and returns this Sankey generator. If *links* is not specified, returns the current links accessor, which defaults to:

```js
function links(graph) {
  return graph.links;
}
```

If *links* is specified as a function, the function is invoked when the Sankey layout is [generated](#_sankey), being passed any arguments passed to the Sankey generator. This function must return an array of links. If *links* is not a function, it must be a constant array of *links*.

Each *link* must be an object with the following properties:

* *link*.source - the link’s source [node](#sankey_nodes)
* *link*.target - the link’s target [node](#sankey_nodes)
* *link*.value - the link’s numeric value

For convenience, a link’s source and target may be initialized using numeric or string identifiers rather than object references; ; see [*sankey*.nodeId](#sankey_nodeId). The following properties are assigned to each link by the [Sankey generator](#_sankey):

* *link*.y0 - the link’s vertical starting position (at source node)
* *link*.y1 - the link’s vertical end position (at target node)
* *link*.width - the link’s width (proportional to *link*.value)
* *link*.index - the zero-based index of *link* within the array of links
* *link*.circular - set to *true* if the link is circular

For any links that circular (*link*.circular = *true*), the following properties are assigned

* *link*.circularLinkID - the zero-based index of *link* within the array of circular links
* *link*.circularLinkType - set to either *top* or *bottom*, which relates to whether the link is drawn above or below the main part of the graph
* *link*.circularLinkPathData - an object containing the properties used to draw the SVG path, including *link*.circularLinkPathData.path, which is the string used for the *d* property.

<a name="sankey_nodeId" href="#sankey_nodeId">#</a> <i>sankey</i>.<b>nodeId</b>([<i>id</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L77 "Source")

If *id* is specified, sets the node id accessor to the specified function and returns this Sankey generator. If *id* is not specified, returns the current node id accessor, which defaults to the numeric *node*.index:

```js
function id(d) {
  return d.index;
}
```

The default id accessor allows each link’s source and target to be specified as a zero-based index into the [nodes](#sankey_nodes) array. For example:

```js
var nodes = [
  {"id": "Alice"},
  {"id": "Bob"},
  {"id": "Carol"}
];

var links = [
  {"source": 0, "target": 1}, // Alice → Bob
  {"source": 1, "target": 2} // Bob → Carol
];
```

Now consider a different id accessor that returns a string:

```js
function id(d) {
  return d.id;
}
```

With this accessor, you can use named sources and targets:

```js
var nodes = [
  {"id": "Alice"},
  {"id": "Bob"},
  {"id": "Carol"}
];

var links = [
  {"source": "Alice", "target": "Bob"},
  {"source": "Bob", "target": "Carol"}
];
```

This is particularly useful when representing graphs in JSON, as JSON does not allow references. See [this example](https://bl.ocks.org/mbostock/f584aa36df54c451c94a9d0798caed35).

<a name="sankey_nodeAlign" href="#sankey_nodeAlign">#</a> <i>sankey</i>.<b>nodeAlign</b>([<i>align</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L81 "Source")

If *align* is specified, sets the node [alignment method](#alignments) the specified function and returns this Sankey generator. If *align* is not specified, returns the current node alignment method, which defaults to [d3.sankeyJustify](#sankeyJustify). The specified function is evaluated for each input *node* in order, being passed the current *node* and the total depth *n* of the graph (one plus the maximum *node*.depth), and must return an integer between 0 and *n* - 1 that indicates the desired horizontal position of the node in the generated Sankey diagram.

<a name="sankey_nodeWidth" href="#sankey_nodeWidth">#</a> <i>sankey</i>.<b>nodeWidth</b>([<i>width</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L85 "Source")

If *width* is specified, sets the node width to the specified number and returns this Sankey generator. If *width* is not specified, returns the current node width, which defaults to 24.

<a name="sankey_nodePadding" href="#sankey_nodePadding">#</a> <i>sankey</i>.<b>nodePadding</b>([<i>padding</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L89 "Source")

If *padding* is specified, sets the vertical separation between nodes at each column to the specified number and returns this Sankey generator. If *padding* is not specified, returns the current node padding, which defaults to 8.

<a name="sankey_nodePaddingRatio" href="#sankey_nodePaddingRatio">#</a> <i>sankey</i>.<b>nodePaddingRatio</b>([<i>proportion</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L89 "Source")

If *proportion* is specified (from 0 to 1), sets the vertical separation between nodes at each column to the specified number and returns this Sankey generator. If *padding* is not specified, returns the current node padding, which defaults to 8.

<a name="sankey_extent" href="#sankey_extent">#</a> <i>sankey</i>.<b>extent</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L105 "Source")

If *extent* is specified, sets the extent of the Sankey layout to the specified bounds and returns the layout. The *extent* bounds are specified as an array \[\[<i>x0</i>, <i>y0</i>\], \[<i>x1</i>, <i>y1</i>\]\], where *x0* is the left side of the extent, *y0* is the top, *x1* is the right and *y1* is the bottom. If *extent* is not specified, returns the current extent which defaults to [[0, 0], [1, 1]].

<a name="sankey_size" href="#sankey_size">#</a> <i>sankey</i>.<b>size</b>([<i>size</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L101 "Source")

An alias for [*sankey*.extent](#sankey_extent) where the minimum *x* and *y* of the extent are ⟨0,0⟩. Equivalent to:

```js
sankey.extent([[0, 0], size]);
```

<a name="sankey_iterations" href="#sankey_iterations">#</a> <i>sankey</i>.<b>iterations</b>([<i>iterations</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L109 "Source")

If *iterations* is specified, sets the number of relaxation iterations when [generating the layout](#_sankey) and returns this Sankey generator. If *iterations* is not specified, returns the current number of relaxation iterations, which defaults to 32.

<a name="sankey_circularLinkGap" href="#sankey_circularLinkGap">#</a> <i>sankey</i>.<b>circularLinkGap</b>([<i>circularLinkGap</i>]) [<>](https://github.com/d3/d3-sankey/blob/master/src/sankey.js#L109 "Source")

If *circularLinkGap* is specified, sets the gap (in pixels) between circular links that travel next to each other. If *circularLinkGap*, it defaults to 2.

### Alignments

See [*sankey*.nodeAlign](#sankey_nodeAlign).

<a name="sankeyLeft" href="#sankeyLeft">#</a> d3.<b>sankeyLeft</b>(<i>node</i>, <i>n</i>) [<>](https://github.com/d3/d3-sankey/blob/master/src/align.js#L7 "Source")

<img alt="left" src="https://raw.githubusercontent.com/d3/d3-sankey/master/img/align-left.png" width="480">

Returns *node*.depth.

<a name="sankeyRight" href="#sankeyRight">#</a> d3.<b>sankeyRight</b>(<i>node</i>, <i>n</i>) [<>](https://github.com/d3/d3-sankey/blob/master/src/align.js#L11 "Source")

<img alt="right" src="https://raw.githubusercontent.com/d3/d3-sankey/master/img/align-right.png" width="480">

Returns *n* - 1 - *node*.height.

<a name="sankeyCenter" href="#sankeyCenter">#</a> d3.<b>sankeyCenter</b>(<i>node</i>, <i>n</i>) [<>](https://github.com/d3/d3-sankey/blob/master/src/align.js#L19 "Source")

<img alt="center" src="https://raw.githubusercontent.com/d3/d3-sankey/master/img/align-center.png" width="480">

Like [d3.sankeyLeft](#sankeyLeft), except that nodes without any incoming links are moved as right as possible.

<a name="sankeyJustify" href="#sankeyJustify">#</a> d3.<b>sankeyJustify</b>(<i>node</i>, <i>n</i>) [<>](https://github.com/d3/d3-sankey/blob/master/src/align.js#L15 "Source")

<img alt="justify" src="https://raw.githubusercontent.com/d3/d3-sankey/master/img/energy.png" width="480">

Like [d3.sankeyLeft](#sankeyLeft), except that nodes without any outgoing links are moved to the far right.

### Links

<a name="sankeyPath" href="#sankeyPath">#</a><b>sankeyPath</b>[<>](https://github.com/d3/d3-sankey/blob/master/src/sankeyLinkHorizontal.js "Source")

For a given link, returns a path d string, dependent on the link.circular property.

If link.circular is *true*, then link.circularPathData.path is returned.

Else, it returns the path generated by [d3-shape horizontal link](https://github.com/d3/d3-shape/blob/master/README.md#linkHorizontal) suitable for a Sankey diagram. The [source accessor](https://github.com/d3/d3-shape/blob/master/README.md#link_source) in this case is defined as:

```js
function source(d) {
  return [d.source.x1, d.y0];
}
```

and the [target accessor](https://github.com/d3/d3-shape/blob/master/README.md#link_target) is defined as:

```js
function target(d) {
  return [d.target.x0, d.y1];
}
```

For example, to render the links of a Sankey diagram in SVG, you could:

```js

svg.append("g")
    .attr("class", "links")
    .attr("fill", "none")
    .attr("stroke-opacity", 0.2)
    .selectAll("path");
    .data(sankey.links)
    .enter()
    .append("path") 
        .attr("d", sankeyPath)
        .style("stroke-width", function (link) { link.width; })
        .style("stroke", function (link, i) {
            return link.circular ? "red" : "black"
        })
```
