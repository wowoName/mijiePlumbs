<template>
    <el-container class="container" id="work-container">

        <el-aside width="200px">
            <div class="box-card">
                <div class="header">组件</div>
                <div class="card-body">
                    <div class="item" v-for="(item,i) in comList" :key="i">
                        <i class="chart-item" :class="item.type" :data-key="item.type">{{item.name}}</i>
                    </div>
                </div>
            </div>
        </el-aside>
        <el-main>
            <div class="workplace" id="workplace">
            </div>
        </el-main>

    </el-container>
</template>

<script>
export default {
    name: "DragToWorkplace",
    data() {
        return {
            list: ["circle", "diamond", "ellipse", "rectangle", "server", "computer"],
            comList: [
                // {
                //   name: "圆",
                //   type: "circle"
                // },
                // {
                //   name: "菱形",
                //   type: "diamond"
                // },
                // {
                //   name: "椭圆",
                //   type: "ellipse"
                // },
                // {
                //   name: "矩形",
                //   type: "rectangle"
                // },
                {
                    name: "服务器",
                    type: "server"
                },
                {
                    name: "电脑",
                    type: "computer"
                },
                {
                    name: "交换机",
                    type: "switches"
                }
            ],
            jsp: null
        };
    },
    mounted() {
        const that = this;
        jsPlumb.ready(function () {
            // 默认配置
            let instance = jsPlumb.getInstance({
                // drag options
                DragOptions: { cursor: "pointer", zIndex: 2000 },
                // overlays
                ConnectionOverlays: [
                    [
                        "PlainArrow", //Arrow PlainArrow
                        {
                            location: 1,
                            visible: true,
                            width: 11,
                            height: 11,
                            id: "Arrow"
                        }
                    ],
                    [
                        "Label",
                        {
                            location: 0.5,
                            id: "label",
                            label: "至",
                            cssClass: "aLabel",
                            events: {
                                tap: function (e) {
                                    var newText = prompt("请输入内容");
                                    if (newText) {
                                        e.setLabel(newText);
                                    }
                                }
                            }
                        }
                    ]
                ],
                Container: "workplace"
            });
            //保存instace
            that.jsp = instance;
            // 连接线默认样式
            let connectorPaintStyle = {
                strokeWidth: 1,
                stroke: "#61B7CF",
                joinstyle: "round",
                outlineStroke: "white",
                outlineWidth: 2
            };
            // 鼠标移动上 连接线样式
            let connectorHoverStyle = {
                strokeWidth: 2,
                stroke: "#00CED1",
                outlineWidth: 2,
                outlineStroke: "white"
            };

            let endpointHoverStyle = {
                fill: "#216477",
                stroke: "#216477"
            };
            // source endpoints
            let sourceEndpoint = {
                endpoint: "Dot",
                paintStyle: {
                    stroke: "#7AB02C",
                    fill: "transparent",
                    radius: 7,
                    strokeWidth: 1
                },
                isSource: true,
                connector: [
                    "Flowchart",
                    {
                        stub: [40, 60],
                        gap: 10,
                        cornerRadius: 5,
                        alwaysRespectStubs: true
                    }
                ],
                connectorStyle: connectorPaintStyle, //连接线的样式
                hoverPaintStyle: endpointHoverStyle,
                connectorHoverStyle: connectorHoverStyle,
                dragOptions: {},
                overlays: [
                    [
                        "Label",
                        {
                            location: [0.5, 1.5],
                            label: "Drag",
                            cssClass: "endpointSourceLabel",
                            visible: false
                        }
                    ]
                ]
            };
            // target endpoints
            let targetEndpoint = {
                endpoint: "Dot",
                paintStyle: { fill: "#7AB02C", radius: 7 },
                hoverPaintStyle: endpointHoverStyle,
                maxConnections: -1,
                dropOptions: { hoverClass: "hover", activeClass: "active" },
                isTarget: true,
                overlays: [
                    [
                        "Label",
                        {
                            location: [0.5, -0.5],
                            label: "Drop",
                            cssClass: "endpointTargetLabel",
                            visible: false
                        }
                    ]
                ]
            };
            //连接线连接完毕修改连接下的名称
            let init = function (connection) {
                connection
                    .getOverlay("label")
                    .setLabel(
                        connection.source.innerHTML + "->" + connection.target.innerHTML
                    );
            };
            let addEndpoints = function (toId, sourceAnchors, targetAnchors) {
                console.log(toId, sourceAnchors, targetAnchors);

                for (var i = 0; i < sourceAnchors.length; i++) {
                    var sourceUUID = toId + sourceAnchors[i];
                    instance.addEndpoint(toId, sourceEndpoint, {
                        anchor: sourceAnchors[i],
                        uuid: sourceUUID
                    });
                }
                for (var j = 0; j < targetAnchors.length; j++) {
                    var targetUUID = toId + targetAnchors[j];
                    instance.addEndpoint(toId, targetEndpoint, {
                        anchor: targetAnchors[j],
                        // anchor: 'Continuous',
                        uuid: targetUUID
                    });
                }
            };

            // 暂停渲染，执行以下操作
            instance.batch(function () {
                // listen for new connections;
                instance.bind("connection", function (connInfo, originalEvent) {
                    init(connInfo.connection);
                });
                /* // make all the window divs draggable
                instance.draggable($(".workplace .chart-item"), {
                  grid: [1, 1]
                }); */

                // listen for clicks on connections, and offer to delete connections on click.
                instance.bind("click", function (conn, originalEvent) {
                    // if (confirm("Delete connection from " + conn.sourceId + " to " + conn.targetId + "?"))
                    //   instance.detach(conn);
                    // conn.toggleType("basic");
                });

                instance.bind("connectionDrag", function (connection) {
                    console.log(
                        "connection " +
                        connection.id +
                        " is being dragged. suspendedElement is ",
                        connection.suspendedElement,
                        " of type ",
                        connection.suspendedElementType
                    );
                });
                //连接线
                instance.bind("connectionDragStop", function (connection) {
                    console.log("connection " + connection.id + " was dragged");
                });

                instance.bind("connectionMoved", function (params) {
                    console.log("connection " + params.connection.id + " was moved");
                });
            });
            // 将模块拖入画板中
            $(".box-card .chart-item").draggable({
                scope: "plant",
                helper: "clone",
                containment: $("#work-container")
            });
            $("#workplace").droppable({
                scope: "plant",
                drop: function (ev, ui) {
                    let id = "item" + new Date().getTime();
                    //获取元素绑定的属性
                    let html = `<div id="${id}" class="chart-item ${ui.helper.attr("data-key")}" >${ui.helper.html()}</div>`;
                    $(this).append(html);
                    $("#" + id).css({
                        top: ui.position.top - 60 + "px",
                        left: ui.position.left - 200 + "px"
                    });
                    addEndpoints(
                        id,
                        ["RightMiddle", "BottomCenter"],
                        ["LeftMiddle", "TopCenter"]
                    );

                    /* jsPlumb.addEndpoint(id, { anchors: "TopCenter" });
                    jsPlumb.addEndpoint(id, { anchors: "RightMiddle" });
                    jsPlumb.addEndpoint(id, { anchors: "BottomCenter" });
                    jsPlumb.addEndpoint(id, { anchors: "LeftMiddle" }); */

                    /* instance.draggable(jsPlumb.getSelector("#" + id), {
                      containment: "parent"
                    }); */
                    instance.draggable(id, {
                        grid: [1, 1]
                        // containment: true
                    });
                    //添加修改组件名称
                    doubleclick("#" + id);
                }
            });

            jsPlumb.fire("jsPlumbDemoLoaded", instance);
            //修改节点名称
            function doubleclick(id) {
                $(id).dblclick(function () {
                    var newText = prompt("请输入内容");
                    if (newText) {
                        $(this).text(newText);
                    }
                });
            }
            //控制连接线的显示
            $("#workplace").on("mouseenter", function () {
                $(".jtk-endpoint").show();
            });
            $("#workplace").on("mouseleave", function () {
                $(".jtk-endpoint").hide();
            });
            //添加移除元素图标
            $("#workplace").on("mouseenter", ".chart-item", function () {
                $(this).append(
                    '<img src="../../static/img/close.png" class="removeCom" style="position: absolute;" />'
                );
                $("img").css({
                    right: 0,
                    top: 5
                });
            });
            $("#workplace").on("mouseleave", ".chart-item", function () {
                $("img").remove();
            });
            //通过移除图标删除u元素
            $("#workplace").on("click", ".removeCom", function () {
                let curid = $(this).parent().attr("id");
                console.log(curid);
                if (confirm("确定要删除吗?")) {
                    // jsPlumb.removeAllEndpoints(curid, true);
                    //$(this).parent().remove();
                    //删除方法 --不同版本之间删除方法有差异
                    //删除元素及其端点
                    instance.remove(curid);
                    //removeAllEndpoints 删除端点保留元素
                    //删除当前元素
                    // $(parentnode).remove();
                }
            });


            //保存
            function saveFlowchart() {
                //获取所有的连接线
                let connections = [];
                $.each(instance.getConnections(), function (idx, connection) {
                    connections.push({
                        sourceId: connection.sourceId,
                        targetId: connection.targetId,
                        uuids: connection.getUuids(),
                        anchors: $.map(connection.endpoints, function (endpoint) {
                            //  console.error(endpoint.getUuid());
                            return [
                                [
                                    endpoint.anchor.x,
                                    endpoint.anchor.y
                                ]
                            ];

                        })
                    });
                });

                //获取所有的元素
                var nodes = []
                $("#workplace .chart-item").each(function (idx, elem) {
                    var $elem = $(elem);
                    nodes.push({
                        blockId: $elem.attr("id"),
                        nodetype: $elem.attr("data-nodetype"),
                        positionX: parseInt($elem.css("left"), 10),
                        positionY: parseInt($elem.css("top"), 10)
                    });
                });
                var flowChart = {};
                flowChart.nodes = nodes;
                flowChart.connections = connections;
                console.log(JSON.stringify(flowChart));
            };

        });
        //添加保存数据/回显数据的方法
        this.addFun();
    },
    methods: {
        addFun() {
            const that = this;
            document.addEventListener("keydown", e => {
                if (e.keyCode == 13)
                    that.saveFlowchart();
                else if (e.keyCode == 32)
                    that.setData();
            })
        },
        //保存数据
        saveFlowchart() {
            //获取所有的连接线
            let connections = [];
            $.each(this.jsp.getConnections(), function (idx, connection) {
                connections.push({
                    sourceId: connection.sourceId,
                    targetId: connection.targetId,
                    uuids: connection.getUuids(),
                    anchors: $.map(connection.endpoints, function (endpoint) {
                        //  console.error(endpoint.getUuid());
                        return [
                            [
                                endpoint.anchor.x,
                                endpoint.anchor.y
                            ]
                        ];

                    })
                });
            });

            //获取所有的元素
            var nodes = []
            $("#workplace .chart-item").each(function (idx, elem) {
                var $elem = $(elem);
                nodes.push({
                    id: $elem.attr("id"),
                    positionX: parseInt($elem.css("left"), 10),
                    positionY: parseInt($elem.css("top"), 10),
                    text: $elem.html()
                });
            });
            var flowChart = {};
            flowChart.nodes = nodes;
            flowChart.connections = connections;
            console.log(JSON.stringify(flowChart));
        },
        // 初始化node节点
        initNode(el) {
            // initialise draggable elements.
            // 元素拖动，基于 katavorio.js 插件
            let _self = this;
            this.jsp.draggable(el, {
                filter: ".resize",
                // containment: true,
                start(params) {
                    // 拖动开始
                    // console.log(params);
                },
                drag(params) {
                    // 拖动中
                    // console.log(params);
                },
                stop(params) {
                    // 拖动结束
                    console.log(params);
                }
            });

            this.jsp.makeSource(el, {
                filter: ".ep",
                // anchor: "Continuous",
                anchor: ["Perimeter", { shape: "Rectangle" }],
                connectorStyle: {
                    stroke: "#5c96bc",
                    strokeWidth: 2,
                    outlineStroke: "transparent",
                    outlineWidth: 4
                },
                extract: {
                    action: "the-action"
                },
                maxConnections: -1,
                onMaxConnections: function (info, e) {
                    alert("Maximum connections (" + info.maxConnections + ") reached");
                }
            });

            this.jsp.makeTarget(el, {
                dropOptions: { hoverClass: "dragHover" },
                anchor: ["Perimeter", { shape: "Rectangle" }],
                allowLoopback: false
            });

            // this is not part of the core demo functionality; it is a means for the Toolkit edition's wrapped
            // version of this demo to find out about new nodes being added.
            //
            this.jsp.fire("jsPlumbDemoNodeAdded", el);
        },
        setData() {
            //清空
            this.jsp.empty("workplace");
            let url = "/static/json/json.3.json";
            this.$axios
                .get(url)
                .then(resp => {
                    this.chartData = resp.data;
                    this.$nextTick(() => {
                        this.chartData.nodes.forEach(item => {
                            this.initNode(item.blockId);
                        });
                        // this.jsp.empty();
                        this.chartData.connections.forEach(item => {
                            this.jsp.connect({
                                source: item.sourceId,
                                target: item.targetId
                            });
                        });
                    });
                })
                .catch(err => {
                    console.log(err);
                });
        }
    }
};
</script>
<style lang="scss">
.workplace {
    width: 100%;
    height: 100%;
    position: relative;
}
@mixin components($url) {
    background: url($url) no-repeat center center / 40px 40px;
    width: 80px;
    height: 90px;
    line-height: 150px;
    font-size: 0.5em;
}
.server {
    @include components("../../static/img/icon_server.png");
}
.computer {
    @include components("../../static/img/icon_virtual_host.png");
}
.switches {
    @include components("../../static/img/icon_router.png");
}
.chart-item:hover {
    box-shadow: 2px 2px 19px #444;
    opacity: 0.8;
    border-radius: 10px;
}
</style>


