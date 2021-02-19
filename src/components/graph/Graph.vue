<template>
  <div class="graph">
    <div class="canvas" ref="canvas"></div>
    <div class="draggable-container">
      <div class="draggable-relative">
        <vue-draggable-resizable
            v-for="element in elementsEditing"
            :key="element.id"
            class="draggable"
            drag-handle=".drag-handle"
            :x="element.getBBox().center().x - 160"
            :y="element.getBBox().bottomLeft().y + 10"
            :w="320"
            h="auto"
            @dragging="onDrag"
            @resizing="onResize"
            :resizable="true"
            :parent="false">
          <element-edit :element="element" @close="closeElementInfo(element)"/>
        </vue-draggable-resizable>
        <vue-draggable-resizable
            v-for="link in linksEditing"
            :key="link.id"
            class="draggable"
            drag-handle=".drag-handle"
            :x="link.getBBox().center().x - 160"
            :y="link.getBBox().topLeft().y + 50"
            :w="320"
            h="auto"
            @dragging="onDrag"
            @resizing="onResize"
            :resizable="true"
            :parent="false">
          <link-edit :link="link" @close="closeLinkInfo(link)"/>
        </vue-draggable-resizable>
      </div>
    </div>
  </div>
</template>

<script type="ts">
import {dia, shapes, elementTools, linkTools, V} from 'jointjs';
import ElementEdit from './ElementEdit';
import LinkEdit from './LinkEdit';
import VueDraggableResizable from 'vue-draggable-resizable/src/components/vue-draggable-resizable'

// TODO: Move one-time setup to plugin
dia.Element.prototype.getHighlighterPath = function (w, h) {
  return ['M', w, 0, w, h, 0, h, 0, 0, 'z'].join(' ');
};
elementTools.InfoButton = class extends elementTools.Button {
  constructor(onClick) {
    super({
      markup: [{
        tagName: 'circle',
        selector: 'button',
        attributes: {
          'r': 7,
          'fill': '#001DFF',
          'cursor': 'pointer'
        }
      }, {
        tagName: 'path',
        selector: 'icon',
        attributes: {
          'd': 'M -2 4 2 4 M 0 3 0 0 M -2 -1 1 -1 M -1 -4 1 -4',
          'fill': 'none',
          'stroke': '#FFFFFF',
          'stroke-width': 2,
          'pointer-events': 'none'
        }
      }],
      x: '100%',
      y: '0',
      offset: {
        x: 0,
        y: 0
      },
      rotate: true,
      action: onClick
    });
  }
}
elementTools.AddButton = class extends elementTools.Button {
  constructor(onClick) {
    super({
      markup: [{
        tagName: 'circle',
        selector: 'button',
        attributes: {
          'r': 7,
          'fill': '#00cd00',
          'cursor': 'pointer'
        }
      }, {
        tagName: 'path',
        selector: 'icon',
        attributes: {
          'd': 'M -4 0 4 0 M 0 -4 0 4',
          'fill': 'none',
          'stroke': '#FFFFFF',
          'stroke-width': 2,
          'pointer-events': 'none'
        }
      }],
      x: '100%',
      y: '100%',
      offset: {
        x: 0,
        y: 0
      },
      rotate: true,
      action: onClick
    });
  }
}
linkTools.InfoButton = linkTools.Button.extend({
  name: 'info-button',
  options: {
    markup: [{
      tagName: 'circle',
      selector: 'button',
      attributes: {
        'r': 7,
        'fill': '#001DFF',
        'cursor': 'pointer'
      }
    }, {
      tagName: 'path',
      selector: 'icon',
      attributes: {
        'd': 'M -2 4 2 4 M 0 3 0 0 M -2 -1 1 -1 M -1 -4 1 -4',
        'fill': 'none',
        'stroke': '#FFFFFF',
        'stroke-width': 2,
        'pointer-events': 'none'
      }
    }],
    distance: 80,
    offset: 0
  }
});

export default {
  name: 'Graph',
  components: {
    ElementEdit,
    LinkEdit,
    VueDraggableResizable
  },
  data() {
    return {
      graph: null,
      paper: null,
      elements: [],
      links: [],
      elementsEditing: [],
      linksEditing: []
    };
  },
  methods: {
    addElement({clone, attr, position, size, graph = this.graph}) {
      let element;
      if (clone)
        element = clone.clone();
      else {
        element = new shapes.standard.Rectangle();
        element.resize(100, 40);
      }
      if (position) {
        if(!Array.isArray(position))
          position = [position.x, position.y];
        element.position(...position);
      }
      if (size)
        element.resize(...size);
      if (attr)
        element.attr(attr);
      element.addTo(graph);
      this.elements.push(element);

      // Add tools
      const elementView = element.findView(this.paper);
      const toolsView = new dia.ToolsView({
        tools: [
          new elementTools.Remove(),
          new elementTools.InfoButton((event) => this.openElementInfo(element, event)),
          new elementTools.AddButton((event) => this.addLink({from: element})),
        ]
      });
      elementView.addTools(toolsView);
      elementView.hideTools();

      return element;
    },
    addLink({from, to, attr, graph = this.graph}) {
      const link = new shapes.standard.Link();
      if (from)
        link.source(from);
      if (to)
        link.target(to);
      else if (from) {
        const point = from.getBBox().bottomRight();
        point.x += 20;
        point.y += 20;
        link.target(point);
      }
      if(attr)
        link.attr(attr);
      link.addTo(graph);
      // link.attr({
      //   '.connection': { stroke: 'black' },
      //   '.marker-source': { fill: 'black', d: 'M 10 0 L 0 5 L 10 10 z' },
      //   '.marker-target': { fill: 'black', d: 'M 10 0 L 0 5 L 10 10 z' }
      // });
      this.links.push(link);

      // Add tools
      const linkView = link.findView(this.paper);
      const toolsView = new dia.ToolsView({
        tools: [
          new linkTools.Remove(),
          new linkTools.InfoButton({action: (event) => this.openLinkInfo(link, event)}),
          new linkTools.SourceArrowhead(),
          new linkTools.TargetArrowhead(),
        ]
      });
      linkView.addTools(toolsView);
      linkView.hideTools();

      console.log("New Link: " + JSON.stringify(link));

      return link;
    },
    isEditingElement(element) {
      return this.elementsEditing.some(e => e === element);
    },
    openElementInfo(element, event) {
      console.log("Element: " + JSON.stringify(element));
      console.log("Event: " + JSON.stringify(event));
      //alert('View id: ' + this.id + '\n' + 'Model id: ' + this.model.id);
      if (!this.isEditingElement(element))
        this.elementsEditing.push(element);
    },
    closeElementInfo(element) {
      console.log("closeElementInfo");
      const index = this.elementsEditing.indexOf(element);
      if (index > -1) {
        this.elementsEditing.splice(index, 1);
      }
    },
    isEditingLink(link) {
      return this.linksEditing.some(l => l === link);
    },
    openLinkInfo(link) {
      //alert('View id: ' + this.id + '\n' + 'Model id: ' + this.model.id);
      if (!this.isEditingLink(link))
        this.linksEditing.push(link);
    },
    closeLinkInfo(link) {
      const index = this.linksEditing.indexOf(link);
      if (index > -1) {
        this.linksEditing.splice(index, 1);
      }
    },
    onDrag(...args) {
      //console.log("Drag: " + JSON.stringify(args));
    },
    onResize(...args) {
      //console.log("Resize: " + JSON.stringify(args));
    },
    removeElement(element) {
      console.log("Remove element: "+JSON.stringify(element));
      this.graph.removeElement(element);
    }
  },
  mounted() {
    this.graph = new dia.Graph;
    this.paper = new dia.Paper({
      el: this.$refs['canvas'],
      model: this.graph,
      width: '100%',
      height: '100%',
      gridSize: 1,
      linkPinning: false,
      highlighting: false,
    });

    // Setup graph
    this.graph.on('change:position', function (cell, value) {
    });
    this.graph.on('change:target', function (cell, value) {
    });
    this.graph.on("remove", (cell, linkedElements, view) => {
      if(this.isEditingElement(cell))
        this.closeElementInfo(cell);
      else if(this.isEditingLink(cell))
        this.closeLinkInfo(cell);
    });

    // Setup paper
    this.paper.on('blank:pointerdblclick', (o, x, y) => {
      this.addElement({position: [x, y]});
    });
    this.paper.on('blank:pointerclick', function (o, x, y) {
      console.log("Blank Click: " + JSON.stringify(o));
    });
    this.paper.on('blank:pointerrightclick', function (o, x, y) {
      console.log("Right Click: " + JSON.stringify(o));
    });
    this.paper.on('blank:pointerup', function (...args) {
      console.log("Pointer Up: " + JSON.stringify(args));
    });
    this.paper.on('blank:contextmenu', function (...args) {
      console.log("Context Menu: " + JSON.stringify(args));
    });
    this.paper.on('element:pointerdblclick', (elementView, x, y) => {
      this.openElementInfo(elementView.model);
    });
    this.paper.on('link:pointerdblclick', function (linkView, x, y) {
      this.openLinkInfo(linkView.model);
    });
    this.paper.on('cell:pointerdblclick', function (cellView, x, y) {
    });
    this.paper.on('element:mouseenter', function (elementView) {
      elementView.showTools();
    });
    this.paper.on('element:mouseleave', function (elementView) {
      elementView.hideTools();
    });
    this.paper.on('link:mouseenter', function (linkView) {
      linkView.showTools();
      console.log("Hover Link: " + console.log(linkView));
    });
    this.paper.on('link:mouseleave', function (linkView) {
      linkView.hideTools();
    });

    // Highlighting
    const highlighter = V('path', {
      'stroke': '#e9fc03',
      'stroke-width': '2px',
      'fill': 'transparent',
      'pointer-events': 'none'
    });
    this.paper.on('cell:highlight', (cellView) => {
      const padding = 5;
      const bbox = cellView.getBBox({useModelGeometry: true}).inflate(padding);

      highlighter.translate(bbox.x, bbox.y, {absolute: true});
      highlighter.attr('d', cellView.model.getHighlighterPath(bbox.width, bbox.height));

      V(this.paper.viewport).append(highlighter);
    });
    this.paper.on('cell:unhighlight', function () {
      highlighter.remove();
    });

    // Create elements
    const rect = this.addElement({
      position: [100, 30],
      size: [100, 40],
      attr: {
        body: {
          fill: '#ffa2a2'
        },
        label: {
          text: 'Hello',
          fill: 'black'
        }
      }
    });
    const rect2 = this.addElement({
      clone: rect,
      position: [400, 30],
      attr: {
        body: {
          fill: '#5fffff'
        },
        label: {
          text: 'HackDays'
        }
      }
    });
    const link = this.addLink({
      from: rect,
      to: rect2,
      attr: {
        line: {
          stroke: '#0000FF',
        }
      }
    });
  }
}
</script>

<style lang="scss" scoped>
.graph {
  position: relative;
  width: 100%;
  height: 100%;

  .canvas {
    width: 100%;
    height: 100%;

    .joint-paper {
      border: 1px solid #A0A0A0;
    }
  }
}

.draggable-container {
  position: absolute;
  top: 0px;
  left: 0px;
  pointer-events: none;
}

.draggable-relative {
  position: relative;

  * {
    pointer-events: auto;
  }
}

.draggable {
  border: none;
}
</style>
