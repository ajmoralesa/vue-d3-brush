<template>
  <div>
    <Button @click="addRandomBrush">Add Random Brush</Button>

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

onMounted(() => {
  d3.select(svgRef.value).call(d3.brushX());
});

const brushes = ref([]);

const addRandomBrush = () => {
  // Calculate two random time points within the domain
  const domain = xz.value.domain();
  const startTime = new Date(
    domain[0].getTime() +
      Math.random() * (domain[1].getTime() - domain[0].getTime())
  );
  const endTime = new Date(
    startTime.getTime() +
      Math.random() * (domain[1].getTime() - startTime.getTime())
  );

  // Create a new brush
  const newBrush = d3.brushX().extent([
    [margin.left, 0],
    [width - margin.right, height],
  ]);

  newBrush.move(d3.select(svgRef.value), [
    xz.value(startTime),
    xz.value(endTime),
  ]);

  // Push the new brush to the brushes array (you should define brushes as a ref)
  brushes.value.push({
    brush: newBrush,
    startTime,
    endTime,
  });

  selectionExtent.value = [startTime, endTime];
};

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

  // Get the brust x0 and x1 values
  const [x0, x1] = selectionExtent.value.map(xz.value);
  brushes.value[0].brush.move(d3.select(svgRef.value), [x0, x1]);
}

onMounted(() => {
  d3.select(svgRef.value).call(zoom).on("dblclick.zoom", null);
});

const selectionExtent = ref([new Date(), new Date()]);
</script>
