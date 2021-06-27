<template>
    <div class="drag ">
        <div class="blockDiv" :style="{width : boxWidth+'px'}">
            <div class="box" v-show="isDrag"
                 :style="{transform: `translate(${x}px,${y}px)` ,width:dragInfo.width,height:dragInfo.height,lineHeight: dragInfo.height,background:dragInfo.color}">
                {{dragInfo.text}}
            </div>
            <div class="block" :ref="'block'+index"
                 :class="[isTargetDrag&&dragIndex ===index? 'isTargetDrag':'']"
                 :style="{background:isDrag && index===Number(dragInfo.index ) ?actInfo.color: item.color,height:dragHeight+'px',lineHeight:dragHeight+'px'}"
                 @mousedown="dragMove($event,index)"
                 v-for="(item,index) in dragAry"
                 :key="index">
                {{isDrag && index===Number(dragInfo.index )?actInfo.text : item.text}}
            </div>
        </div>
    </div>
</template>

<script>

    export default {
        name: 'drag',
        props: {
            boxWidth: {
                type: Number,
                default: 500
            },
            dragHeight: {
                type: Number,
                default: 50
            },
            dragAry: {
                type: Array,
                default:[]
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
                    text: '',
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
                ev.preventDefault()
                const {color, text} = this.dragAry[index]
                const {clientX, clientY} = ev
                const {offsetLeft, offsetTop, offsetWidth, offsetHeight, parentElement} = ev.target
                const dx = clientX - offsetLeft, dy = clientY - offsetTop
                let moveX, moveY
                this.isDrag = true
                this.x = offsetLeft
                this.y = offsetTop
                this.dragInfo.width = offsetWidth + 'px'
                this.dragInfo.height = offsetHeight + 'px'
                this.dragInfo.index = index
                this.dragInfo.text = text
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
                            this.actInfo.color = this.dragAry[indexs].color
                            this.actInfo.text = this.dragAry[indexs].text
                            this.dragIndex = indexs
                            this.isTargetDrag = true
                        }
                    })

                    this.x = moveX
                    this.y = moveY
                }
                document.onmouseup = () => {
                    if (this.isTargetDrag) {
                        const tempIndex = index, temp = this.dragAry[tempIndex]
                        this.$set(this.dragAry, tempIndex, this.dragAry[this.dragIndex])
                        this.$set(this.dragAry, this.dragIndex, temp)
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
        line-height: 50px;
        color: #fff;
        position: absolute;
        cursor: move;
        transition-duration: 100ms;
        transition-timing-function: ease-out;
    }

    .active {
        border: 1px dashed #000;
    }

    .blockDiv {
        width: 500px;
        display: flex;
        margin: 0 auto;
        flex-wrap: wrap;
        position: relative;
        transition-duration: 500ms;
        transition-timing-function: ease-out;
    }

    .block {
        width: calc(calc(100% / 3) - 10px);
        margin: 5px;
        height: 50px;
        line-height: 50px;
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
