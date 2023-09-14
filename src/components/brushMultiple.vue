<template>
  <div>
    <Button @click="removeAllBrushes">Remove all Brushes</Button>

    <svg
      ref="svgRef"
      :height="height"
      :width="width"
      style="background-color: lightblue"
    >
      <g>
        <g ref="axisBottomRef" :transform="axisBottomTransform"></g>
      </g>

      <text class="axis-title" x="50%" y="90%" dy="2.5em" text-anchor="middle">
        Time (mm:ss)
      </text>
    </svg>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, watchEffect } from "vue";
import { scaleTime } from "d3-scale";
import { axisBottom } from "d3-axis";
import { select } from "d3-selection";
import * as d3 from "d3";

const margin = reactive({
  top: 80,
  right: 10,
  bottom: 50,
  left: 80,
});

const width = 600 - margin.left - margin.right;
const height = 400 - margin.top - margin.bottom;

const svgRef = ref();

const scaleX = computed(() => {
  return scaleTime()
    .domain([
      new Date("2000-01-01T00:00:01.000Z"),
      new Date("2000-01-01T00:35:53.000Z"),
    ])
    .range([margin.left, width - margin.right]);
});

const axisBottomRef = ref();

onMounted(() => {
  watchEffect(() => {
    if (!axisBottomRef.value) return;
    select(axisBottomRef.value).call(axisBottom(scaleX.value));
  });
});

//////////////////// Brushing functionality /////////////////////////

const gBrushes = ref();
const brushes = ref([]);
const xBrus0 = ref();
const xBrus1 = ref();

onMounted(() => {
  gBrushes.value = d3.select(svgRef.value).append("g").attr("class", "brushes");
  newBrush();
  drawBrushes();
});

function newBrush() {
  var newBrush = d3
    .brushX()
    .on("brush end", ({ selection }) => {
      const [x0, x1] = selection;
      xBrus0.value = scaleX.value.invert(x0);
      xBrus1.value = scaleX.value.invert(x1);
      newBrush.xValue = [xz.value(x0), xz.value(x1)];
      newBrush.x0 = xBrus0;
      newBrush.x1 = xBrus1;
    })
    .on("end", brushend);
  brushes.value.push({
    id: brushes.value.length,
    brush: newBrush,
    start: xBrus0.value,
    end: xBrus1.value,
  });
  drawBrushes();
}

function brushend() {
  var lastBrushID = brushes.value[brushes.value.length - 1].id;
  var lastBrush = document.getElementById("brush-" + lastBrushID);
  var selection = d3.brushSelection(lastBrush);

  if (selection && selection[0] !== selection[1]) {
    newBrush();
  }
  drawBrushes();
}

function drawBrushes() {
  var brushSelection = gBrushes.value
    .selectAll(".brush")
    .data(brushes.value, function (d) {
      return d.id;
    });

  // Remove any brushes that are no longer in the data
  brushSelection.exit().remove();

  brushSelection
    .enter()
    .insert("g", ".brush")
    .attr("class", "brush")
    .attr("id", function (brush) {
      return "brush-" + brush.id;
    })
    .each(function (brushObject) {
      var brushGroup = d3.select(this);
      brushObject.brush(brushGroup);
    });

  brushSelection.each(function (brushObject) {
    d3.select(this)
      .attr("class", "brush")
      .selectAll(".overlay")
      .style("pointer-events", function () {
        var brush = brushObject.brush;
        if (
          brushObject.id === brushes.value.length - 1 &&
          brush !== undefined
        ) {
          return "all";
        } else {
          return "none";
        }
      });
  });
}

//// Zooming functionality ////
const zoom = d3
  .zoom()
  .extent([
    [margin.left, 0],
    [width - margin.right, height],
  ])
  .translateExtent([
    [margin.left, -Infinity],
    [width - margin.right, Infinity],
  ])
  .filter((event) => {
    return event.type === "wheel";
  })
  .on("zoom", zoomed);

// When zooming, redraw the area and the x axis.
const xz = ref(scaleX.value);

function zoomed(event) {
  // Update xz
  xz.value = event.transform.rescaleX(scaleX.value);
  d3.select(axisBottomRef.value).call(axisBottom(xz.value));

  // Iterate through all the brushes and update their extents
  brushes.value.forEach((brushItem) => {
    const [x0, x1] = [brushItem.start, brushItem.end].map(xz.value);
    brushItem.brush.move(d3.select(`#brush-${brushItem.id}`), [x0, x1]);
  });
}

onMounted(() => {
  d3.select(svgRef.value).call(zoom).on("dblclick.zoom", null);
});

function removeAllBrushes() {
  brushes.value = []; // Empty the brushes array
  drawBrushes(); // Redraw the (empty) brushes
  newBrush(); // Add a new brush
}
</script>
