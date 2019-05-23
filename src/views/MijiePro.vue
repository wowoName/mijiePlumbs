<template>
    <el-container class="container" id="work-container">
        <!-- <el-aside width="200px">
            <div class="template-box">
                <div class="header">我的模板
                    <i class="el-icon-circle-plus-outline add" title="新建" @click="handleClickTemp()"></i>
                </div>
                <ul class="template-list">
                    <li class="item" :class="{'active':item.isActive}" v-for="(item,index) in templateList" :key="item.key" @click="handleClickTemp(item.key,index)">
                        {{item.name}}
                    </li>
                </ul>
            </div>
        </el-aside> -->

        <el-main>
            <div class="toolCon">
                <div class="save" title="保存" @click="saveFlowchart"></div>
                <div class="save" title="回显" @click="handleClickTemp('init',1)"></div>
                <div class="clear" title="清空" @click="clearFlowchart"></div>
            </div>
            <div class="workplace" id="workplace">
                <template v-for="(item, idx) in chartData.nodes">
                    <mijie-group v-if="item.type === 'groups'" v-bind="item" :key="idx" @resize="resizeGroup"></mijie-group>
                    <mijie-node v-else v-bind="item" :key="idx"></mijie-node>
                </template>
            </div>
        </el-main>
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

        <el-dialog title="" :visible.sync="showInfos" width="600px">
            <el-form ref="serverForm" :model="form" :rules="rules" label-width="80px" size="small">

                <el-form-item label="服务地址" prop="url">
                    <el-input v-model="form.url" placeholder="请输入服务地址"></el-input>
                </el-form-item>

                <el-form-item label="appID" prop="app">
                    <el-input v-model="form.app" placeholder="请输入appID名"></el-input>
                </el-form-item>

                <el-form-item label="实例ID" prop="instancId">
                    <el-input v-model="form.instancId" placeholder="请输入实例ID"></el-input>
                </el-form-item>
                <el-form-item label="主机名" prop="hostName">
                    <el-input v-model="form.hostName" placeholder="主机名"></el-input>
                </el-form-item>

                <!-- <el-form-item label="状态" prop="status">
                    <el-switch v-model="form.status">
                    </el-switch>
                </el-form-item> -->

                <el-form-item label="状态" prop="status">
                    <el-select v-model="form.status" placeholder="请选择">
                        <el-option label="UP" value="UP"></el-option>
                        <el-option label="DOWN" value="DOWN"></el-option>
                        <el-option label="STARTING" value="STARTING"></el-option>
                        <el-option label="OUT_OF_SERVICE" value="OUT_OF_SERVICE"></el-option>
                        <el-option label="UNKNOWN" value="UNKNOWN"></el-option>
                    </el-select>
                </el-form-item>
                <!-- <el-form-item>
                    <el-button type="primary" @click="onSubmit">立即创建</el-button>
                    <el-button>取消</el-button>
                </el-form-item> -->
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="showInfos = false">取 消</el-button>
                <el-button type="primary" @click="setAttr">确 定</el-button>
            </span>
        </el-dialog>
    </el-container>

</template>

<script>
import MijieNode from "@/components/MijieNode";
import MijieGroup from "@/components/MijieGroup";
export default {
    name: "DragToWorkplace",
    components: {
        MijieNode,
        MijieGroup
    },
    data() {
        return {
            showInfos: false,//设置属性
            editId: "",//修改节点的id
            form: {
                url: "",
                app: "",
                instancId: "",
                hostName: "",
                status: ""
            }, rules: {
                url: [
                    { required: true, message: '请输入服务地址', trigger: 'blur' },
                    // { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
                ],
                app: [
                    { required: true, message: '请输入server名', trigger: 'blur' }
                ],
                instancId: [
                    { type: 'string', required: true, message: '请输入实例ID', trigger: 'blur' }
                ],
                hostName: [
                    { type: 'string', required: true, message: '请输入主机名', trigger: 'blur' }
                ]

            },
            chartData: {
                nodes: [],
                connections: []
            },
            templateList: [
                {
                    name: "测试模板",
                    key: "test"
                }
            ],
            comList: [
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
                },
                {
                    name: "组",
                    type: "groups"
                }
            ],
            jsp: null,
            jsplulmb: null,
            // source endpoints
            sourceEndpoint: {
                endpoint: "Dot",
                paintStyle: {
                    stroke: "#7AB02C",
                    fill: "transparent",
                    radius: 7,
                    strokeWidth: 1
                },
                isSource: true,//端点类型 起点
                //  Connector: "Straight",// Bezier（贝塞尔曲线），Straight（直线），Flowchart（流程图），StateMachine（状态机）
                connector: [
                    "Flowchart",
                    {
                        stub: [40, 0],
                        gap: 10,//连线端点距离元素距离
                        cornerRadius: 5,//折线处弧度
                        alwaysRespectStubs: true,
                        dashstyle: "2 2"
                    }

                    // "Straight"//直线

                    //    ["Bezier", { curviness: 63 }],

                    // "StateMachine"
                ],
                connectorStyle: {
                    strokeWidth: 1,
                    stroke: "#61B7CF",//连接线的样式
                    joinstyle: "round",
                    outlineStroke: "white",
                    outlineWidth: 2
                }, //连接线的样式
                hoverPaintStyle: {
                    fill: "#216477",
                    stroke: "#216477"
                },
                connectorHoverStyle: {
                    strokeWidth: 2,
                    stroke: "#00CED1",
                    outlineWidth: 2,
                    outlineStroke: "white"
                },
                dragOptions: { cursor: "pointer", zIndex: 2000 },
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
            },
            // target endpoints
            targetEndpoint: {
                endpoint: "Dot",
                paintStyle: { fill: "#7AB02C", radius: 7 },
                hoverPaintStyle: {
                    fill: "#216477",
                    stroke: "#216477"
                },
                connectorStyle: {
                    strokeWidth: 1,
                    stroke: "#61B7CF",
                    joinstyle: "round",
                    outlineStroke: "white",
                    outlineWidth: 2,
                    dashstyle: '2 4'
                },
                maxConnections: -1,
                dropOptions: { hoverClass: "hover", activeClass: "active" },
                isTarget: true,//端点类型  终点
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
            },
            // 公共样式
            commonEndpoint: {
                endpoint: "Dot",
                paintStyle: { fill: "#7AB02C", radius: 7 },
                HoverPaintStyle: { stroke: "#1e8151", strokeWidth: 4, outlineStroke: 'red' },
                isSource: true,
                //scope: "blue",
                connector: [
                    "Flowchart"
                ],
                connectorStyle: {
                    strokeWidth: 2,
                    stroke: "#61B7CF",
                    joinstyle: "round",
                    outlineStroke: "white",
                    outlineWidth: 2
                },
                isTarget: true,
                beforeDrop: function (params) {
                    return confirm(
                        "Connect " + params.sourceId + " to " + params.targetId + "?"
                    );
                },
                dragOptions: { cursor: "pointer", zIndex: 2000 }
            }
        };
    },
    created() {

    },
    mounted() {
        const that = this;
        jsPlumb.ready(function () {
            // 默认配置
            let instance = jsPlumb.getInstance({
                // drag options
                DragOptions: that.sourceEndpoint.dragOptions,
                Endpoint: [
                    "Dot",//端点的类型
                    { cssClass: "chart-dot", hoverClass: "", radius: 5 }
                ],
                connector: that.sourceEndpoint.connector,
                // "dashstyle": "2 4"
                paintStyle: { strokeWidth: 2, stroke: "#32CD32" },//实线 p小写
                // overlays
                HoverPaintStyle: { stroke: "#1e8151", strokeWidth: 4 },
                ConnectionOverlays: [
                    [
                        "Arrow", //Arrow PlainArrow
                        {
                            location: 1,
                            visible: true,
                            width: 8,
                            height: 10,
                            id: "Arrow"
                        }
                    ],
                    // [
                    //     "Label",
                    //     {
                    //         location: 0.5,
                    //         id: "label",
                    //         label: "",
                    //         cssClass: "aLabel",
                    //         events: {
                    //             tap: function (e) {
                    //                 var newText = prompt("请输入内容");
                    //                 if (newText) {
                    //                     e.setLabel(newText);
                    //                 }
                    //             }
                    //         }
                    //     }
                    // ]
                ],
                Container: "workplace"
            });



            //保存instace
            that.jsp = instance;
            //连接线连接完毕修改连接下的名称
            let init = function (connection) {
                connection.getOverlay("label").setLabel(connection.source.innerHTML + "->" + connection.target.innerHTML);
            };
            // 暂停渲染，执行以下操作
            instance.batch(function () {
                // 连线
                instance.bind("connection", function (connInfo, originalEvent) {
                    // init(connInfo.connection);
                });
                /* // make all the window divs draggable
                instance.draggable($(".workplace .chart-item"), {
                  grid: [1, 1]
                }); */

                // listen for clicks on connections, and offer to delete connections on click.
                instance.bind("click", function (conn, originalEvent) {
                    console.log("点击了我");

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
            //可以拖动画布
            // $("#workplace").draggable();
            $("#workplace").droppable({
                scope: "plant",
                drop: function (ev, ui) {
                    let helper = ui.helper,
                        id = new Date().getTime().toString(),//jsPlumbUtil.uuid(),
                        item = {
                            id,
                            type: helper.attr("data-key"),
                            nodeStyle: {
                                top: ui.position.top + "px",
                                left: ui.position.left - 200 + "px"
                            },
                            data: {
                                "app": "SERVER",
                                "hostName": "172.20.10.1",
                                "instanceId": "172.20.10.6:Server:8101",
                                "client": [],
                                "url": "172.20.10.6:8101",
                                "status": "UP"
                            },
                            hostName: helper.html()
                        };
                    //给组添加属性
                    if (item.type === "groups") {
                        item.nodeStyle.width = "200px";
                        item.nodeStyle.height = "200px";
                        item.name = "组";
                    }

                    //将数据存储
                    that.chartData.nodes.push(item);
                    //添加修改组件名称
                    that.$nextTick(() => {
                        //添加组
                        if (item.type === "groups") {
                            that.initGroup(id);
                            jsPlumb.fire("jsPlumbDemoGroupAdded", instance);
                        } else {//添加其他元素
                            /**
                             * 添加连接点 连接点类型
                             * Static
                             *jsPlumb有九个默认位置，元素的四个角，元素的中心，元素的每个边的中点。
                             *Top(TopCenter),TopRight,TopLeft
                             *Right(RightMiddle)
                             *Bottom(BottomCenter),BottomRight,BottomLeft
                             *Left(LeftMiddle)
                             *center
                             */
                            that.initNode(id);
                            that.addEndpoints(id);
                        }
                    });

                }
            });

            jsPlumb.fire("jsPlumbDemoLoaded", instance);

            $("#workplace").on("dblclick", ".chart-item", function () {
                that.editName("#" + $(this).attr("id"));
            });
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
        });

    },
    methods: {
        /**
      * @description 模板点击事件
      * @param {String} key 模板关键值
      */
        handleClickTemp(key, index) {
            this.chartData = {
                nodes: [],
                connections: [],
                props: {}
            };
            let clientSource = ["BottomCenter", "TopCenter"],
                clientTarget = ["LeftMiddle", "RightMiddle"];

            //清空
            this.jsp.empty("workplace");
            if (key) {
                // this.templateList.forEach(item => {
                //     item.isActive = item.key == key;
                // });
                // let curData = this.templateList[index];
                // curData.isActive = true;
                // this.$set(this.templateList, index, curData);
                //加载对应的模板
                let url = "/static/json/" + key + ".json",
                    loading = this.loadingData();
                this.$axios.get(url).then(resp => {
                    let data = resp.data.data,
                        nodes = [],
                        amount = 0,
                        connections = [];//连接点
                    //这是所有的节点
                    data.forEach((v, i) => {
                        //获取组元素
                        let groups = {
                            "id": "groups_" + this.generateUUID(),//生成唯一的groupID 解决清空之后id依然存在的问题
                            "nodeStyle": {
                            },
                            "type": "groups",
                            "data": {},
                            "name": v.region
                        }
                        amount++;
                        let nodesChildren = [];
                        //获取组内的元素
                        v.zone.forEach(item => {
                            //获取server
                            item.servers.forEach(itemSer => {
                                let curId = "computer_" + amount,
                                    node = Object.assign(itemSer, {
                                        "id": curId,
                                        "nodeStyle": {
                                            "width": "90px",
                                            "height": "90px"
                                        },
                                        "type": itemSer.app == "SERVER" ? "server" : "computer",
                                        "data": {
                                            url: itemSer.url,
                                            app: itemSer.app,
                                            instancId: itemSer.instanceId,
                                            hostName: itemSer.hostName + "(" + item.zoneName + ")",
                                            status: itemSer.status
                                        }
                                    });
                                nodesChildren.push(node);
                                amount++;
                                //获取连接点
                                itemSer.client.forEach(itemCli => {
                                    let cliId = "server_" + amount,
                                        node = Object.assign(itemCli, {
                                            "id": cliId,
                                            "nodeStyle": {
                                                "width": "90px",
                                                "height": "90px"
                                            },
                                            "type": itemCli.app == "SERVER" ? "server" : "computer",
                                            "data": {
                                                url: itemCli.url,
                                                app: itemCli.app,
                                                instancId: itemCli.instanceId,
                                                hostName: itemCli.hostName,
                                                status: itemCli.status
                                            }
                                        });
                                    nodesChildren.push(node);
                                    //获取连接点
                                    connections.push({
                                        "sourceId": curId,
                                        "targetId": cliId,
                                        "uuids": [
                                            curId + clientSource[Math.floor(Math.random() * (2 - 0) + 0)],
                                            cliId + clientTarget[Math.floor(Math.random() * (2 - 0) + 0)]
                                        ],
                                    });
                                    amount++;
                                });
                            });
                        });
                        //存入元素
                        groups.children = nodesChildren;
                        nodes.push(groups);
                    });
                    //修改坐标点
                    this.chartData.nodes = this.modifyPo(nodes);
                    this.chartData.connections = connections;
                    this.$nextTick(() => {
                        this.chartData.nodes.forEach(item => {
                            //渲染节点
                            if (item.type !== "groups") {
                                this.initNode(item.id);
                                //添加连接点
                                this.addEndpoints(item.id);
                            } else {
                                this.initGroup(item.id);
                                //渲染组的子元素
                                if (item.hasOwnProperty("children")) {
                                    item.children.forEach(v => {
                                        this.initNode(v.id);
                                        this.addEndpoints(v.id);
                                        //添加到组
                                        //this.addToGroup(item.id.toString(), v.id);
                                    })
                                }
                            }
                        });
                        //连接元素
                        this.chartData.connections.forEach(item => {
                            this.jsp.connect({
                                source: item.sourceId,
                                target: item.targetId,
                                //连接到哪一个点 的什么位置
                                uuids: item.uuids// 添加连接处使用
                            });
                        });
                        //将元素添加到组（避免初次渲染完毕 拖动元素拖动到组外）
                        this.$nextTick(() => {
                            this.chartData.nodes.forEach(item => {
                                //渲染组的子元素
                                if (item.hasOwnProperty("children")) {
                                    item.children.forEach(v => {
                                        this.addToGroup(item.id.toString(), v.id);
                                    })
                                }
                            });
                            //关闭加载状态
                            this.$nextTick(() => {
                                setTimeout(() => {
                                    loading.close();
                                }, 1000);
                            })
                        })



                    });

                }).catch(err => {
                    console.log(err);
                    loading.close();
                });
            }
        },
        //添加到组
        addToGroup(groupID, id) {
            this.jsp.addToGroup(groupID, document.getElementById(id));
        },
        /** 
         * 修改组内元素的坐标避免重叠
        */
        modifyPo(nodes) {
            nodes.forEach(item => {
                //修改group的位置
                let level = Math.ceil(item.children.length / 3),//一行三个可以排几列
                    _height = level * 90 + (level - 1) * 100;//高度为每个元素的高90 +  50间距
                item.nodeStyle = {
                    "top": "10px",
                    "left": "10px",
                    "width": item.children.length >= 3 ? "500px" : (item.children.length * 90 + 100 + level * 100) + "px",
                    "height": level > 1 ? (level * 90 + 100 + (level - 1) * 100) + "px" : "190px"//高度+300 确保连线在组内
                };
                //组内的元素一行放置三个 每一个间隔为50 + 元素本身高度90
                item.children.forEach((v, i) => {
                    let index = i + 1;
                    let _top = (Math.ceil(index / 3) - 1) * 190,
                        _left = index % 3 === 0 ? 330 : (index % 3 - 1) * 140 + 50;

                    v.nodeStyle = {
                        "top": _top + 50 + "px",//top+50 确保连线在组内
                        "left": _left + "px",//left+50 确保连线在组内
                        "width": "90px",
                        "height": "90px"
                    }
                });
            });
            this.modifyGroupPo(nodes);
            return nodes;
        },
        //修改组的位置
        modifyGroupPo(nodes) {
            nodes.forEach((item, i) => {
                //修改group的位置
                let index = i - 2,
                    leftIndex = i % 3,
                    _top = 50,
                    _left = 10;
                if (i > 0) {
                    let indexLeft = nodes[i - 1].nodeStyle.width.indexOf("p"),
                        hassetTop = (Math.ceil(index / 2) - 1);
                    //当时第二排的时候需要设置高度
                    if (i > 1) {
                        let indexTop = nodes[i - 2].nodeStyle.top.indexOf("p"),
                            indexH = nodes[i - 2].nodeStyle.height.indexOf("p");
                        _top = Number(nodes[i - 2].nodeStyle.top.substring(0, indexTop)) + 100;
                        _top += Number(nodes[i - 2].nodeStyle.height.substring(0, indexH));
                    }
                    if (i % 2 != 0)
                        _left = Number(nodes[i - 1].nodeStyle.width.substring(0, indexLeft)) + 100;
                }
                item.nodeStyle.top = _top + "px";
                item.nodeStyle.left = _left + "px";

            });
            return nodes;
        },
        /** 
         * 整理数据格式  拆分成模块以及连线
        */
        renderNodes() {

        },
        //缩放界面
        zoom(scale) {
            $("#workplace").css({
                "-webkit-transform": `scale(${scale})`,
                "-moz-transform": `scale(${scale})`,
                "-ms-transform": `scale(${scale})`,
                "-o-transform": `scale(${scale})`,
                "transform": `scale(${scale})`
            })
            this.jsp.setZoom(0.75);
        },
        //修改节点名称
        editName(id) {
            this.showInfos = true;
            this.$nextTick(() => {
                //情况表单内容
                this.$refs["serverForm"].resetFields();
                let curData = JSON.parse($(id).attr("data-datas"));
                this.form = Object.assign(this.form, curData);
                this.editId = id;
            })

            // var newText = prompt("请输d入内容");
            // if (newText) {
            //     $(id).text(newText);
            // }
        },
        //修改节点属性完毕
        setAttr() {
            this.$refs["serverForm"].validate((valid) => {
                if (valid) {
                    $(this.editId).attr("data-datas", JSON.stringify(this.form));
                    $(this.editId).text(this.form.url);
                    this.showInfos = false;
                    this.$message({
                        message: '配置成功',
                        type: 'success'
                    });
                } else {
                    this.$message({
                        message: '请完善内容',
                        type: 'info'
                    });
                    return false;
                }
            });


        },
        //保存数据
        saveFlowchart() {
            //获取所有的连接线
            let connections = [],
                that = this;
            $.each(this.jsp.getConnections(), function (idx, connection) {
                connections.push({
                    sourceId: connection.sourceId,
                    targetId: connection.targetId,
                    uuids: connection.getUuids(),//连接线
                    anchors: $.map(connection.endpoints, function (endpoint) {
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
            let nodes = [];
            //排除组内的元素
            $("#workplace>.chart-item").each(function (idx, elem) {
                let $elem = $(elem),
                    text = $elem.html(),
                    type = $elem.attr("data-type");
                let curData = {
                    id: $elem.attr("id"),
                    nodeStyle: {
                        top: parseInt($elem.css("top"), 10) + "px",
                        left: parseInt($elem.css("left"), 10) + "px",
                        width: parseInt($elem.css("width"), 10) + "px",
                        height: parseInt($elem.css("height"), 10) + "px"
                    },
                    text: text,
                    type: type,//   type/className
                    data: JSON.parse($elem.attr("data-datas"))//当前节点的私有属性
                };
                if (type == "groups") {
                    curData.text = $elem.find("title").html();
                    curData.children = that.getGroupChildren($elem.find(".chart-item"));
                }
                nodes.push(curData);

            });
            var Flowchart = {};
            Flowchart.nodes = nodes;
            Flowchart.connections = connections;
            console.log(JSON.stringify(Flowchart));
        },
        //获取组的子元素
        getGroupChildren(data) {
            let nodes = [];
            data.each(function (idx, elem) {
                let $elem = $(elem),
                    text = $elem.html(),
                    type = $elem.attr("data-type");
                let curData = {
                    id: $elem.attr("id"),
                    nodeStyle: {
                        top: parseInt($elem.css("top"), 10) + "px",
                        left: parseInt($elem.css("left"), 10) + "px",
                        width: parseInt($elem.css("width"), 10) + "px",
                        height: parseInt($elem.css("height"), 10) + "px"
                    },
                    text: text,
                    type: type,//   type/className
                    data: JSON.parse($elem.attr("data-datas"))//当前节点的私有属性
                };
                nodes.push(curData);
            });

            return nodes;
        },
        //清空数据
        clearFlowchart() {
            this.jsp.empty("workplace");
            this.chartData = {
                nodes: [],
                connections: []
            };
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
                    // console.log(params, "开始");
                },
                drag(params) {
                    // 拖动中
                    //  console.log(params, "拖动ing...");
                },
                stop(params) {
                    // 拖动结束
                    //  console.log(params, "拖动结束");
                }
            });

            // this is not part of the core demo functionality; it is a means for the Toolkit edition's wrapped
            // version of this demo to find out about new nodes being added.
            //
            this.jsp.fire("jsPlumbDemoNodeAdded", el);
        },
        //初始化组
        initGroup(id) {
            this.jsp.addGroup({
                el: document.getElementById(id), // 必须为 dom对象
                id: id,
                droppable: true,
                constrain: true,
                dropOverride: false,
                dropOptions: {
                    drop: function (p) {
                        console.log(p);
                        return true;
                    }
                },
                orphan: false
            });
        },
        //动态设置连接线的样式
        getPointStyle(type) {
            let styles = {
                endpoint: "Dot",
                paintStyle: { fill: "#7AB02C", radius: 7 },
                //  isSource: true,
                //scope: "blue",
                connector: [
                    "Flowchart",
                    {
                        stub: [30, false, 1000, 0, 0],//线段拐点高度
                        gap: 0,//连线端点距离元素距离
                        cornerRadius: 5,//折线处弧度
                        alwaysRespectStubs: true,
                        dashstyle: "2 2"
                    }

                    //"Straight"//直线

                    //["Bezier", { curviness: 63 }],

                    //"StateMachine"
                ],
                maxConnections: 3,//每个端点最多连接的数量
                onMaxConnections(params, originalEvent) {//当连接数超过的回调
                    alert("每个端点最多连接3个");
                },
                connectorStyle: {
                    strokeWidth: 2,
                    stroke: "#61B7CF",
                    joinstyle: "round",
                    outlineStroke: "transparent",
                    outlineWidth: 2
                },
                // isTarget: true,
                beforeDrop: function (params) {
                    //当连接是进行判断
                    console.log("是这里的吧");
                    return true
                    // return confirm(
                    //     "Connect " + params.sourceId + " to " + params.targetId + "?"
                    // );
                },
                dragOptions: { cursor: "pointer", zIndex: 2000 }
            };
            if (type == "source") { //起点
                styles.isSource = true;
                //修改起点的样式
                styles.paintStyle = {
                    stroke: "#7AB02C",
                    fill: "transparent",
                    radius: 7,
                    strokeWidth: 1
                };
                delete styles.beforeDrop;
            } else {//终点
                styles.isTarget = true;
            }
            return styles;
        },
        //添加连接节点
        addEndpoints(toId, sourceAnchors = ["BottomCenter", "TopCenter"], targetAnchors = ["LeftMiddle", "RightMiddle"]) {
            const that = this;
            //绘制连线的起点
            for (let i = 0; i < sourceAnchors.length; i++) {
                let sourceUUID = toId + sourceAnchors[i],
                    styles = that.getPointStyle("source");
                styles.connectorStyle.stroke = "rgb(" + Math.round(Math.random() * 255) + "," + Math.round(Math.random() * 255) + ',' + Math.round(Math.random() * 10) + ')';
                //给特定的节点 设置连线的样式（联系的样式 在source 点设置）
                if (sourceAnchors[i] == "TopCenter") {
                    styles.connectorStyle.dashstyle = "2 4";
                }
                //uuid 连接线使用
                that.jsp.addEndpoint(toId, styles, {
                    anchor: sourceAnchors[i],
                    uuid: sourceUUID
                });
            }
            //绘制连线的终点
            for (let j = 0; j < targetAnchors.length; j++) {
                let targetUUID = toId + targetAnchors[j];
                that.jsp.addEndpoint(toId, {
                    anchor: targetAnchors[j],
                    uuid: targetUUID
                }, that.getPointStyle("target"));
            }
        },
        /**
         * @description 组的拖拽
         * @param {Object} item 节点当前数据
         * @param {Number} idx 下标
         */
        resizeGroup(el) {
            console.log(el);
            this.$nextTick(() => {
                this.jsp.repaintEverything();
            });
        },
        //生成唯一标识符
        generateUUID() {
            var d = new Date().getTime();
            if (window.performance && typeof window.performance.now === "function") {
                d += performance.now(); //use high-precision timer if available
            }
            var uuid = 'xxxxxxxxLxxxxL4xxxLyxxxLxxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
                var r = (d + Math.random() * 16) % 16 | 0;
                d = Math.floor(d / 16);
                return (c == 'x' ? r : (r & 0x3 | 0x8)).toString(16);
            });
            return uuid;
        },
        //请求加载状态
        loadingData() {
            return this.$loading({
                lock: true,
                text: '加载中...',
                spinner: 'el-icon-loading',
                background: 'rgba(0, 0, 0, 0.7)'
            });
        }
    }
};
</script>
<style lang="scss" >
@mixin components($url, $w: 90px, $h: 90px, $bw: 40px) {
    background: url($url) no-repeat center center / $bw $bw;
    width: $w;
    height: $h;
    line-height: 150px;
    font-size: 0.5em;
    cursor: pointer;
}

.workplace {
    width: 100%;
    height: 100%;
    position: relative;
    overflow: auto;
}
.toolCon {
    position: absolute;
    top: 2px;
    right: 202px;
    width: 80px;
    z-index: 100;
    padding: 6px 10px;
    display: flex;
    align-items: center;
    justify-content: space-around;
    border-radius: 3px;
    background-color: #545c64;
    .save {
        @include components("../../static/img/save.png", 22px, 22px, 16px);
    }
    .clear {
        @include components("../../static/img/clear.png", 22px, 22px, 16px);
    }
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
.groups {
    @include components("../../static/img/bg_edit.png");
}
.card-body-pro {
    padding: 10px 0;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
}
</style>


