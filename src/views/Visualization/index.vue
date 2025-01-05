<template>
  <div class="Visualization">
    <a-card class="card-0">
      <div class="init0" ref="init0"></div>
    </a-card>
  </div>
</template>
<script setup>
import * as echarts from "echarts";
import { POST } from "@/utils/axios";
import { textAreaProps } from "ant-design-vue/es/input/inputProps";

const init0 = ref(null);
const Init0 = async () => {
  const { code, data } = await POST("/testproject/group/name");
  if (code != 0) return;
  const resultList = data.map((i) => {
    return {
      name: i.name,
      value: i.testproject,
    };
  });
  const myChart = echarts.init(init0.value);
  const option = {
    title: {
      text: "检测项目统计",
      left: "center",
    },

    toolbox: {
      show: false,
      feature: {
        mark: { show: true },
        dataView: { show: true, readOnly: false },
        restore: { show: true },
        saveAsImage: { show: true },
      },
    },
    series: [
      {
        // name: 'Nightingale Chart',
        type: "pie",
        radius: [50, 200],
        center: ["50%", "50%"],
        roseType: "area",
        itemStyle: {
          borderRadius: 8,
        },
        data: resultList,
      },
    ],
  };
  myChart.setOption(option);
};

onMounted(() => {
  Init0();
});
</script>
<style lang="scss" scoped>
.Visualization {
  width: 100%;
  display: flex;
  flex-wrap: wrap;

  .card-0 {
    width: calc(50% - 5px);
    height: 800px;
    margin-right: 5px;
    margin-bottom: 5px;
    :deep(.ant-card-body) {
      height: 100%;
      .init0 {
        height: 100%;
      }
    }
  }
}
</style>
