---
title: Topic Map
---
# Topic Map

<div id="svg">
</div>

<div id="graph-dot-text" style="display: none">
digraph
{
  prelim [label="Preliminaries", shape=box]
  compileModel [label="C/C++ Compilation Model"]
  linkage [label="Linkage"]
  headers [label="Headers"]
  buildSystems [label="Build Systems"]
  unitTesting [label="Unit Testing"]
  classes [label="Classes", shape=box]
  methods [label="Methods", shape=box]
  properties [label="Properties", shape=box]
  inheritance [label="Inheritance", shape=box]
  virtualMethods [label="Virtual Methods", shape=box]
  opOverloading [label="Operator Overloading", shape=box]
  interfaces [label="Interfaces", shape=box]
  templates [label="Template Basics", shape=box]
  refCounting [label="Reference Counting", shape=box]
  exceptions [label="Exceptions", shape=box]
  stl [label="Standard Template Library", shape=box]
  designPatterns [label="Design Patterns", shape=box]
  decorator [label="Decorator"]
  factory [label="Factory"]
  adapter [label="Adapter"]
  composite [label="Composite"]
  command [label="Command"]
  oopImpl [label="Implementation of OOP", shape=box]
  langSurvey [label="OOP Language Survey", shape=box]
  c [label="C"]
  java [label="Java"]
  cSharp [label="C#"]
  javascript [label="Javascript"]
  
  prelim -> { compileModel, linkage, headers, buildSystems, unitTesting }
         -> classes 
         -> { methods, inheritance }
  inheritance -> virtualMethods
  methods -> { virtualMethods, properties, opOverloading }
  { properties, virtualMethods } -> interfaces
  interfaces -> { designPatterns, oopImpl, langSurvey }
  classes -> templates 
  templates -> { refCounting, stl }
  refCounting -> exceptions -> designPatterns
  stl -> designPatterns
  opOverloading -> refCounting
  designPatterns -> { decorator, factory, adapter, composite, command }
  langSurvey -> { c, java, cSharp, javascript }
}
</div>

<script src="libs/viz.js">
</script>
<script>
  var graphText = document.getElementById("graph-dot-text").innerText;

  var svg = Viz(graphText, "svg");
  
  var parser = new DOMParser();
  var dom = parser.parseFromString(svg, "text/xml");
  document.getElementById("svg").appendChild(dom.documentElement);
</script>