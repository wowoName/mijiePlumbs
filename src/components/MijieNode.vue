<template>
    <!-- type=className  -->
    <div class="chart-item" :class="type" :style="nodeStyle" :data-type="type" :data-datas="JSON.stringify(data)" :id="id" @dblclick="handleNodeDbclick">
        {{hostName}}
    </div>
</template>
<script>
export default {
    name: "MijieNode",
    // props 验证
    props: {
        id: {
            type: String,
            required: true
        },
        hostName: {
            type: String,
            required: true
        },
        type: {
            type: String,
            required: true
        },
        nodeStyle: {
            type: Object
        },
        data: {
            type: Object
        }
    },
    data() {
        return {};
    },
    methods: {
        handleNodeDbclick() {
            // this.$emit("edit");
        }
    }
};
</script>
<style lang="scss">
</style>


