---
title: 'Smart Home'
layout: category
taxonomy: smart-home
entries_layout: grid
classes: wide
---
<script src="https://unpkg.com/gojs/release/go-debug.js"></script>
<div id="diagram"
     style="width:100%; height:500px; background-color: #DAE4E4;"></div>
 <script>
     var myDiagram = new go.Diagram("diagram",
        {
        "undoManager.isEnabled": true,
        layout: new go.TreeLayout({ angle: 90, layerSpacing: 35 })
        });

        myDiagram.nodeTemplate =
        new go.Node("Horizontal",
        { background: "#44CCFF" })
        .add(new go.Picture(
            { margin: 10, width: 50, height: 50, background: "red" })
            .bind("source"))
        .add(new go.TextBlock("Default Text",
            { margin: 12, stroke: "white", font: "bold 16px sans-serif" })
            .bind("text", "name"));

        // define a Link template that routes orthogonally, with no arrowhead
        myDiagram.linkTemplate =
        new go.Link(
            // default routing is go.Link.Normal
            // default corner is 0
            { routing: go.Link.Orthogonal, corner: 5 })
            // the link path, a Shape
            .add( new go.Shape({ strokeWidth: 3, stroke: "#555" }))
            // if we wanted an arrowhead we would also add another Shape with toArrow defined:
            //.add( new go.Shape({  toArrow: "Standard", stroke: null  }))

        // it's best to declare all templates before assigning the model
        myDiagram.model = new go.TreeModel(
        [
            { key: "1",              name: "Home Assistant Blue", source: "none.png"},
            { key: "2", parent: "1", name: "deCONZ",    source: "none.png" },
            { key: "3", parent: "1", name: "MQTT",   source: "none.png" },
            { key: "4", parent: "3", name: "MiLight Hub", source: "none.png" },
            { key: "5", parent: "4", name: "MiLight Bulbs",     source: "none.png" },
            { key: "6", parent: "2", name: "Gledopto Light Bulbs", source: "none.png" },
            { key: "6", parent: "2", name: "Aqara Sensors", source: "none.png" }
        ]);
 </script>    