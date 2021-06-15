<template>
  <div id="app">
    <x-json-view
      :show-depth="-1"
      :json-data="jsonData"
      :is-show-err-notice="false"
      @handleJsonView="handleJsonView"
    >
      <template slot="open">+</template>
      <template slot="close">-</template>
    </x-json-view>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      jsonData: '{ "creator": { "creator1": 123, "creator2": [ { "creator21": 234 } ], "creator3": 456 }, "dataSwitch": { "dataSwitch1": 234 }, "dataSwitchCn": "开", "eventDesc": "Click", "eventId": 289, "eventName": "Click", "hasReportedData": true, "hasReportedDataCn": "是", "isHide": false, "isHideCn": "显示", "lastEditTime": "2021-06-09 15:04:19", "lastEditor": "王伟", "lastEditorUserId": "4378", "metaEventType": "preset", "metaEventTypeCn": "预置", "projectId": 2, "recentlyDataNum": 0, "remark": "", "status": "display", "statusCn": "正常" }',
      a: {},
    };
  },
  methods: {
    handleJsonView(data) {
      console.log(data);
    },
  },
};
</script>
