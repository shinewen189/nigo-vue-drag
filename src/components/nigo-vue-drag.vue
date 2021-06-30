<template>
    <div class="drag">
        <div :style="{width : boxWidth==='auto'? 'auto': boxWidth+'px'}"
             :class="getmode"
        >
            <div :style="{transform: `translate(${x}px,${y}px)` ,
            width:dragInfo.width,height:dragInfo.height,
            background:dragInfo.color}"
                 class="box"
                 v-show="isDrag" v-html="dragAry[dragInfo.index].html">
            </div>
            <div :class="[
                setBlcock,isTargetDrag&&dragIndex ===index? 'isTargetDrag':'',
            !dragMode&&isDrag&&dragInfo.index ===index?'active':'',
            isTargetDrag&&dragIndex ===index&&dragMode?'dragModeAct':'']"
                 :key="index"
                 :ref="'block'+index"
                 :style="{background:dragMode&&isDrag && index===Number(dragInfo.index) ?actInfo.color: item.color ,
                 height:dragHeight==='auto'?'auto':dragHeight+'px'}"
                 @mousedown.prevent="dragMove($event,index)"
                 v-for="(item,index) in dragAry"
                 v-html="dragMode&&isDrag && index===Number(dragInfo.index)?actInfo.html :item.html"
            >
            </div>
        </div>
    </div>

</template>

<script>

    export default {
        name: 'drag',
        props: {
            //元素排序模式   1:change |0:insert
            dragMode: {
                type: [Number, Boolean],
                default: 1
            },
            //元素布局模式   flex |list
            mode: {
                type: String,
                default: 'flex'
            },
            //盒子宽度  'auto'| number
            boxWidth: {
                type: [Number, String],
                default: 'auto'
            },
            //拖拽元素高度   'auto'| 自定义高度
            dragHeight: {
                type: [Number,String],
                default: 50
            },
            //传入的元素数组    [color : '',html : '']
            dragAry: {
                type: Array,
                default:[]
            }
        },
        computed: {
            // 计算排列模式
            getmode() {
                return this.mode === 'list' ? 'blockList' : 'blockFlex'
            },
            // 计算排列模式
            setBlcock() {
                return this.mode === 'list' ? 'lblock' : 'fblock'
            }
        },
        data() {
            return {
                x: 0,  //拖拽的x坐标
                y: 0,  // 拖拽的y坐标
                isDrag: false,//是否在拖拽
                //正在拖拽元素的信息
                dragInfo: {
                    width: '',
                    height: '',
                    background: '',
                    index: 0
                },
                //目标元素信息
                actInfo: {
                    color: '',
                    text: ''
                },
                //是否目标元素
                isTargetDrag: false,
                //目标元素索引值
                dragIndex: null,
            }
        },
        methods: {
            //拖拽逻辑
            dragMove(ev, index) {
                const {color} = this.dragAry[index]
                const {clientX, clientY} = ev
                const {offsetLeft, offsetTop, offsetWidth, offsetHeight, parentElement} = ev.currentTarget
                const dx = clientX - offsetLeft, dy = clientY - offsetTop
                let moveX, moveY
                this.isDrag = true
                this.x = offsetLeft
                this.y = offsetTop
                this.dragInfo.width = offsetWidth + 'px'
                this.dragInfo.height = offsetHeight + 'px'
                this.dragInfo.index = index
                this.dragInfo.color = color
                document.onmousemove = (moveEv) => {
                    moveEv.preventDefault()
                    moveX = moveEv.clientX - dx
                    moveY = moveEv.clientY - dy
                    if (moveX < 0) {
                        moveX = 0
                    }
                    if (moveX > parentElement.offsetWidth - offsetWidth) {
                        moveX = parentElement.offsetWidth - offsetWidth
                    }
                    if (moveY < 0) {
                        moveY = 0
                    }
                    if (moveY > parentElement.offsetHeight - offsetHeight) {
                        moveY = parentElement.offsetHeight - offsetHeight
                    }
                    this.dragAry.forEach((item, indexs) => {
                        const [{offsetLeft: ox, offsetTop: oy}] = this.$refs['block' + indexs]
                        if (moveX + offsetWidth / 2 > ox && moveX + offsetWidth / 2 < ox + offsetWidth &&
                            moveY + offsetHeight / 2 > oy && moveY + offsetHeight / 2 < oy + offsetHeight) {
                            this.dragIndex = indexs
                            this.isTargetDrag = true
                            this.actInfo.html = this.dragAry[indexs].html
                            this.actInfo.color = this.dragAry[indexs].color
                        }
                    })

                    this.x = moveX
                    this.y = moveY
                }
                document.onmouseup = () => {
                    if (this.isTargetDrag) {
                        const tempIndex = index, temp = this.dragAry[tempIndex]
                        if (this.dragMode) {
                            this.$set(this.dragAry, tempIndex, this.dragAry[this.dragIndex])
                            this.$set(this.dragAry, this.dragIndex, temp)
                        } else {
                            this.dragAry.splice(this.dragIndex + 1, 0, temp)
                            this.dragAry.splice(tempIndex, 1)
                        }
                        this.$emit('dragMouseup', {index, dragIndex: this.dragIndex})
                    }

                    this.isDrag = false
                    this.isTargetDrag = false
                    document.onmousemove = null
                    document.onmousedown = null
                }
            }
        }
    }
</script>

<style scoped>
    .box {
        color: #fff;
        position: absolute;
        cursor: move;
        transition-duration: 100ms;
        transition-timing-function: ease-out;
        z-index: 111111;
        background: red;
    }

    .active {
        background: #fff !important;
        border: 1px dashed #000;
    }

    .blockFlex {
        width: 500px;
        display: flex;
        margin: 0 auto;
        flex-wrap: wrap;
        position: relative;
        transition-duration: 500ms;
        transition-timing-function: ease-out;
        overflow: hidden;

    }

    .fblock {
        width: calc(calc(100% / 3) - 10px);
        margin: 5px;
        height: 50px;
        color: #fff;
        box-sizing: border-box;
        background: red;
        cursor: move;
        transition-duration: 500ms;
        transition-timing-function: ease;
        overflow: hidden;
    }

    .blockList {
        position: relative;
        margin: 0 auto;
        transition-duration: 500ms;
        transition-timing-function: ease-out;
    }

    .lblock {
        width: 100%;
        height: 50px;
        margin-bottom: 20px;
        color: #fff;
        box-sizing: border-box;
        background: red;
        cursor: move;
        transition-duration: 500ms;
        transition-timing-function: ease;
    }


    .isTargetDrag {
        cursor: move;
        transform: scale(1.1);
        transition-duration: 500ms;
        transition-timing-function: ease-in;
        position: relative;
    }
    .isTargetDrag:before{
        border: 1px dashed red;
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
    }

    .dragModeAct {
        background: #fff !important;
    }
</style>
