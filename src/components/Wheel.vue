<script setup lang="ts">
import * as d3 from "d3";
import { computed, onMounted, ref, watch } from "vue";

const props = withDefaults(
  defineProps<{
    animDuration: number;
    data: any;
    gift: number;
    imgSrc: any;
    isClicked: boolean;
  }>(),
  {
    animDuration: 6000,
    data: {},
    imgSrc: "",
  }
);

const emit = defineEmits<{
  (e: "done"): void;
}>();

const arrow = ref(null);
const clicked = ref(false);
const container = ref(null);
const pie = ref(null);
const rotation = ref(0);
const style = ref({
  width: 600,
  height: 600,
});
const margin = ref(20);
const svg = ref(null);
const vis = ref(null);
const isLoaded = ref(false);
const interpolate = ref();
const wheelStyle = ref();

watch(
  () => props.isClicked,
  (next, previous) => {
    if (next) {
      spin();
    }
  }
);

const createWheel = () => {
  const wheelSize = computed(() => {
    const screenWidth = document.getElementById("wheel")?.clientWidth;

    const width = Math.min(screenWidth, style.value.width) - margin.value;
    const height = Math.min(screenWidth, style.value.width) + 120;

    return {
      width,
      height,
    };
  });

  wheelStyle.value = {
    width: `${wheelSize.value.width}px`,
    height: `${wheelSize.value.height}px`,
  };

  const rayon = ref(
    Math.min(wheelSize.value.width, wheelSize.value.height) / 2
  );
  // Create Svg
  createSvg(wheelSize.value);
  // Create shadow filter
  createDefs();
  // Create Group container
  createVis();
  // Declare an arc generator function
  createArc(rayon.value);
  // Add the text
  addText();
  // Make circle
  createMiddleCircle(rayon.value);
  createBorderCircle(wheelSize.value);
  // Add img
  if (props.imgSrc) {
    addImgOnCenter(rayon.value);
  }

  // create arrow
  createArrow(wheelSize.value);
};

const createSvg = (wheelSize) => {
  svg.value = d3
    .select("#wheel")
    .append("svg")
    .attr("font-size", "0.6rem")
    .attr("height", "100%")
    .attr("width", "100%")
    .attr(
      "viewBox",
      `0 0 ${parseInt(wheelSize.width + margin.value * 2)} ${parseInt(
        wheelSize.height
      )}`
    )
    .data([props.data])
    .append("g")
    .attr("class", "wrapper")
    .attr(
      "transform",
      `translate(${parseInt(wheelSize.width + margin.value * 2) / 2}, ${
        parseInt(wheelSize.height) / 2
      })`
    );
};

const createDefs = () => {
  const defs = svg.value
    .append("defs")
    .append("filter")
    .attr("id", "shadow")
    .attr("x", "-100%")
    .attr("y", "-100%")
    .attr("width", "550%")
    .attr("height", "550%");

  defs
    .append("feOffset")
    .attr("in", "SourceAlpha")
    .attr("dx", 0)
    .attr("dy", 0)
    .attr("result", "offsetOut");

  defs
    .append("feGaussianBlur")
    .attr("stdDeviation", "9")
    .attr("in", "offsetOut")
    .attr("result", "drop");

  defs
    .append("feColorMatrix")
    .attr("in", "drop")
    .attr("result", "color-out")
    .attr("type", "matrix")
    .attr(
      "values",
      `0 0 0 0   0
          0 0 0 0   0
          0 0 0 0   0
          0 0 0 .3 0
        `
    );

  defs
    .append("feBlend")
    .attr("in", "SourceGraphic")
    .attr("in2", "color-out")
    .attr("mode", "normal");
};

const createVis = () => {
  container.value = svg.value.append("g").attr("class", "wheelholder");

  // Create a G on container
  vis.value = container.value.append("g");

  pie.value = d3
    .pie()
    .value(() => {
      return 1;
    })
    .padAngle(0.01)
    .sort(null);
};

const createArc = (rayon) => {
  const arc = d3.arc().outerRadius(rayon).innerRadius(0);

  // Create the donut slices and also the invisible arcs for the text
  vis.value
    .selectAll(".middleArcText")
    .data(pie.value)
    .enter()
    .append("path")
    .attr("class", "slice")
    .attr("d", arc)
    .attr("stroke", "#000000")
    .attr("stroke-width", "3")
    .attr("fill", (d, i) => {
      return d.data.bgColor;
    })
    .each(function (d, i) {
      const firstArcSection = /(^.+?)L/;

      let newArc = firstArcSection
        .exec(d3.select(this).attr("d"))[1]
        .replace(/,/g, " ");

      if (d.endAngle > (90 * Math.PI) / 180) {
        const startLoc = /M(.*?)A/;
        const middleLoc = /A(.*?)0 0 1/;
        const endLoc = /0 0 1 (.*?)$/;
        const newStart = endLoc.exec(newArc)[1];
        const newEnd = startLoc.exec(newArc)[1];
        const middleSec = middleLoc.exec(newArc)[1];

        newArc = `M${newStart}A${middleSec}0 0 0${newEnd}`;
      }

      vis.value
        .append("path")
        .attr("class", "hiddenarcs")
        .attr("id", "middleArc" + i)
        .attr("d", newArc)
        .style("fill", "none");
    });
};

const addText = () => {
  vis.value
    .selectAll(".middleArcText")
    .data(pie.value)
    .enter()
    .append("text")
    .attr("class", "middleArcText")
    .attr("dy", (d) => {
      return d.endAngle > (90 * Math.PI) / 180 ? -35 : 42;
    })
    .append("textPath")
    .attr("startOffset", "50%")
    .attr("text-anchor", "middle")
    .attr("fill", (d, i) => {
      return d.data.color;
    })
    .attr("xlink:href", (d, i) => {
      return "#middleArc" + i;
    })
    .text((d) => {
      return d.data.value;
    });
};

const createMiddleCircle = (rayon) => {
  container.value
    .append("circle")
    .attr("cx", 0)
    .attr("cy", 0)
    .attr("r", rayon / 2.5)
    .attr("fill", "#ffffff")
    .attr("filter", "url(#shadow)")
    .attr("stroke-width", 4)
    .attr("stroke", "#000000");
};

const createBorderCircle = (wheelSize) => {
  container.value
    .append("g")
    .append("circle")
    .attr("cx", 0)
    .attr("cy", 0)
    .attr("r", (wheelSize.width - 4) / 2)
    .attr("fill", "transparent")
    .attr("stroke-width", "8")
    .attr("filter", "url(#shadow)")
    .attr("stroke", "#ffffff");
};

const addImgOnCenter = (rayon) => {
  container.value
    .append("svg:image")
    .attr("x", `-${rayon / 4}`)
    .attr("y", `-${rayon / 4}`)
    .attr("width", rayon / 2)
    .attr("height", rayon / 2)
    .attr("xlink:href", props.imgSrc);
};

const createArrow = (wheelSize) => {
  const pathArrow =
    "M95.3,9.8c-16.5,0-23.7,15.6-21.9,27c3.4,21.7,21.9,42.2,21.9,42.2s18.5-20.5,21.9-42.2 C118.9,25.4,111.8,9.8,95.3,9.8z";

  arrow.value = container.value
    .append("g")
    .append("path")
    .attr("d", pathArrow)
    .attr("transform", "translate(-20, -300)")
    .attr("stroke", "#ffffff")
    .attr("fill", "#FFFFFF")
    .attr("filter", "url(#shadow)")
    .attr(
      "transform",
      `matrix(1, 0, 0, 1, -95, -${wheelSize.height / 2 - margin.value})`
    )
    .attr("stroke-linejoin", "round")
    .attr("stroke-width", "4");
};

const findCurrentSlice = (index) => {
  return props.data.findIndex((x) => x.id === index) + 1;
};

const spin = async () => {
  if (!clicked.value) {
    clicked.value = true;

    // Define current gain
    const slicedGift = findCurrentSlice(props.gift);
    const dataLength = props.data.length;
    const sliceWidth = 360 / dataLength;
    const currentAngle = 360 - sliceWidth * (slicedGift - 1);
    const numberOfRotation = 360 * 5;
    const rotationn = currentAngle + numberOfRotation;

    rotation.value = Math.round(rotationn / sliceWidth) * sliceWidth;

    let picked = Math.round(dataLength - (rotation.value % 360) / sliceWidth);
    picked = picked >= dataLength ? picked % dataLength : picked;

    // Center slice
    const sliceSize = sliceWidth + sliceWidth / 2;
    rotation.value += sliceSize - Math.round(sliceWidth * 2);

    interpolate.value = d3.interpolate(0, rotation.value);

    const animateVis = () => {
      return vis.value
        .transition()
        .duration(props.animDuration)
        .ease(d3.easeBackOut.overshoot(0.3))
        .attrTween("transform", animRotation)
        .end();
    };

    await animateVis();

    emit("done", props.data[picked]);
  }
};

const animRotation = () => {
  return (t) => {
    return `rotate(${interpolate.value(t)})`;
  };
};

onMounted(() => {
  isLoaded.value = true;
  if (isLoaded.value) {
    createWheel();
  }
});
</script>

<template>
  <div
    id="wheel"
    :class="['wheel', `wheel_font-size--${props.data.length}`]"
    :style="wheelStyle"
  />
</template>


<style>
.wheel {
  width: 100%;
  height: auto;
  margin: 0 auto;
}
.wheel textPath {
  letter-spacing: 1px;
  stroke: rgb(0 0 0 / 10%);
}
.wheel_font-size--1,
.wheel_font-size--2,
.wheel_font-size--3,
.wheel_font-size--4 {
  font-size: 16px;
}
.wheel_font-size--5,
.wheel_font-size--6 {
  font-size: 12px;
}
.wheel_font-size--7,
.wheel_font-size--8 {
  font-size: 10px;
}
</style>
