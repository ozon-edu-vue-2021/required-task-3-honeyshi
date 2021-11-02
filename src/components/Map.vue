<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div v-if="!isLoading" class="map-root">
      <MapSVG ref="svg" v-click-outside="clickOutsideTable" />

      <TableSVG v-show="false" ref="table" />
    </div>
    <div v-else>Loading...</div>
  </div>
</template>

<script>
import MapSVG from "@/assets/images/map.svg";
import TableSVG from "@/assets/images/workPlace.svg";
import legend from "@/assets/data/legend.json";
import tables from "@/assets/data/tables.json";

import * as d3 from "d3";
import ClickOutside from "vue-click-outside";

export default {
  components: {
    MapSVG,
    TableSVG,
  },
  props: {
    handleTableClick: Function,
    handleClickOutside: Function,
  },
  data() {
    return {
      isLoading: false,
      svg: null,
      g: null,
      tables: [],
      tableSVG: null,
    };
  },
  mounted() {
    this.svg = d3.select(this.$refs.svg);
    this.g = this.svg.select("g");
    this.tableSVG = d3.select(this.$refs.table);

    this.tables = tables;

    if (this.g) {
      this.drawTables();
    } else {
      console.log("No group");
    }
  },
  methods: {
    drawTables() {
      // создаем группу рабочих мест
      const svgTableGroups = this.g.append("g").classed("groupPlaces", true);

      const tableClickHandler = this.handleTableClick;

      this.tables.map((table) => {
        // создать группу для рабочего стола
        const svgTable = svgTableGroups
          .append("g")
          .attr("transform", `translate(${table.x}, ${table.y}) scale(0.5)`)
          .attr("id", table._id)
          .classed("employer-place", true)
          .on("click", function () {
            tableClickHandler(this.id);
          });

        svgTable
          .append("g")
          .attr("transform", `rotate(${table.rotate || 0})`)
          .attr("group_id", table.group_id)
          .html(this.tableSVG.html())
          .attr(
            "fill",
            legend.find((it) => it.group_id === table.group_id)?.color ??
              "transparent"
          );
      });
    },
    clickOutsideTable() {
      this.handleClickOutside();
    },
  },
  directives: {
    ClickOutside,
  },
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0px;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>
