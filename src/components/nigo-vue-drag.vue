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
            <div :class="[setBlcock,isTargetDrag&&dragIndex ===index? 'isTargetDrag':'',!dragMode&&isDrag&&dragInfo.index ===index?'active':'']"
                 :key="index"
                 :ref="'block'+index"
                 :style="{background:dragMode&&isDrag && index===Number(dragInfo.index) ?actInfo.color: item.color ,
                 height:dragHeight+'px'}"
                 @mousedown.prevent="dragMove($event,index)"
                 v-for="(item,index) in dragAry"
                 v-html="item.html"

            >
            </div>
        </div>
    </div>
</template>

<script>

    export default {
        name: 'drag',
        props: {
            // 1:change |0:insert
            dragMode: {
                type: Number,
                default: 0
            },
            mode: {
                type: String,
                default: 'flex'
            },
            boxWidth: {
                type: [Number, String],
                default: 'auto'
            },
            dragHeight: {
                type: Number,
                default: 50
            },
            dragAry: {
                type: Array,
                default() {
                    return Array.from({length: 10}, (_, index) => {
                        return {
                            text: index,
                            color: 'red'
                        }
                    })
                }
            }
        },
        computed: {

            getmode() {
                return this.mode === 'list' ? 'blockList' : 'blockFlex'
            },
            setBlcock() {
                return this.mode === 'list' ? 'lblock' : 'fblock'
            }
        },
        data() {
            return {
                x: 0,
                y: 0,
                isDrag: false,
                dragInfo: {
                    width: '',
                    height: '',
                    background: '',
                    index: 0
                },
                actInfo: {
                    color: '',
                    text: ''
                },
                isTargetDrag: false,
                dragIndex: null,
            }
        },
        methods: {
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
                            this.actInfo.text = this.dragAry[indexs].text
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
        overflow: hidden;
        transition-duration: 100ms;
        transition-timing-function: ease-out;
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
        background: #fff !important;
        border: 1px dashed red;
        cursor: move;
    }
</style>
