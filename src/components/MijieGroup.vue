<template>
    <div class="workplace-group " :id="id" :data-datas="JSON.stringify({})" :group="id" :style="nodeStyle" :data-type="type">
        <span class="title">{{name}}</span>
        <div class="ep">
        </div>
        <div class="content">
            <!-- 渲染分区下的主机 -->
            <template v-for="item in children">
                <div v-if="item.type=='line'" :key="item.id">这是一条线</div>
                <mijie-node v-else v-bind="item" :key="item.id"></mijie-node>
            </template>
        </div>
        <!-- 去除缩放 -->
        <!-- <div class="resize top" @mousedown.stop="resize($event, 'top')"></div>
        <div class="resize left" @mousedown.stop="resize($event, 'left')"></div>
        <div class="resize bottom" @mousedown.stop="resize($event, 'bottom')"></div>
        <div class="resize right" @mousedown.stop="resize($event, 'right')"></div> -->

    </div>
</template>
<script>
import MijieNode from "@/components/MijieNode";
import Lisu from "@/components/lisu";
export default {
    name: "ChartNode",
    components: {
        MijieNode,
        Lisu
    },
    // props 验证
    props: {
        id: {
            type: String,
            required: true
        },
        name: {
            type: String
        },
        type: {
            type: String,
            required: true
        },
        icon: {
            type: String
        },
        nodeStyle: {
            type: Object
        },
        children: {
            type: Array,
            default: () => {
                return []
            }
        }
    },
    data() {
        return {
            isMove: false,
        };
    },
    mounted() {

    },
    methods: {
        /**
         * 缩放回调函数
         * @param {Event} e 
         * @param {str} 方向 left right top bottom
         */
        resize(e, type) {
            e.preventDefault();
            let _self = this
            this.isMove = true;

            //设置最小宽度。
            const minWidth = 200;
            const minHeight = 240;

            //获得父元素对象。
            let parent = e.target.parentNode;
            let pWidth = parent.offsetWidth
            let pHeight = parent.offsetHeight
            let top = parseInt(parent.style.top);
            let left = parseInt(parent.style.left);

            //获得当前鼠标位置。
            let mouseX = e.clientX;
            let mouseY = e.clientY;

            document.onmousemove = function (docEvent) {
                if (_self.isMove) {
                    //获得鼠标移动后的坐标。
                    let currentMouseX = docEvent.clientX;
                    let currentMouseY = docEvent.clientY;

                    //计算出移动后的坐标。
                    let moveWidth = currentMouseX - mouseX;
                    let moveHeight = currentMouseY - mouseY;

                    //计算宽高增加或减少的值，增加的量等于鼠标移位的量。
                    let width;
                    let height;

                    switch (type) {
                        case 'top':
                            height = pHeight - moveHeight;
                            if (height > minHeight) {
                                parent.style.top = top + moveHeight + "px";
                                parent.style.height = height + "px";
                            }
                            break;
                        case 'right':
                            width = pWidth + moveWidth;
                            if (width > minWidth) {
                                parent.style.width = width + "px";
                            }
                            break;
                        case 'bottom':
                            height = pHeight + moveHeight;
                            if (height > minHeight) {
                                parent.style.height = height + "px";
                            }
                            break;
                        case 'left':
                            width = pWidth - moveWidth;
                            if (width > minWidth) {
                                parent.style.left = left + moveWidth + "px";
                                parent.style.width = width + "px";
                            }
                            break;

                        default:
                            break;
                    }
                    _self.$emit('resize', parent)


                }
            }
            document.onmouseup = function (event) {
                _self.isMove = false;
                document.onmousemove = function () { return; }
            }
        },
    }
};
</script>
<style lang="scss">
.workplace-group {
    margin: 0;
    cursor: pointer;
    border: 1px solid #ccc;
    position: absolute;
    padding-top: 24px;
    // min-width: 200px;
    // min-height: 250px;
    border-radius: 6px;
    margin-bottom: 20px;

    .title {
        position: absolute;
        top: -30px;
        left: 50%;
        padding: 3px 20px;
        border-radius: 3px;
        transform: translate(-50%);
        background-color: #43cd80;
        color: #ffffff;
    }
    .ep {
        opacity: 0;
        position: absolute;
        right: 0;
        top: 0;
        width: 10px;
        height: 10px;
        background: #409eff;
        border-radius: 5px;
    }
    .content {
        width: 100%;
        height: 100%;
    }
    &:hover {
        .ep {
            opacity: 1;
        }
        .workplace-chart .ep {
            opacity: 0;
        }
    }
    &.dragHover {
        .ep {
            opacity: 0;
        }
        .workplace-chart .ep {
            opacity: 0;
        }
    }
    .workplace-chart:hover .ep {
        opacity: 0;
    }
    .resize {
        width: 8px;
        height: 8px;
        background-color: #ddd;
        border: 1px solid #000;
        position: absolute;
        &.left {
            top: 50%;
            left: -4px;
            cursor: ew-resize;
        }
        &.right {
            top: 50%;
            right: -4px;
            cursor: ew-resize;
        }
        &.top {
            top: -4px;
            left: 50%;
            margin-left: -4px;
            cursor: ns-resize;
        }
        &.bottom {
            bottom: -4px;
            left: 50%;
            margin-left: -4px;
            cursor: ns-resize;
        }
    }
}
</style>


