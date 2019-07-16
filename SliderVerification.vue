/**
* @disabled 传入组件的属性disabled，是否禁用该组件，默认false
* @move 父组件可监听的移动事件，传递给父组件的是一个数字，x轴移动的距离
* @success 验证通过回调方法，返回组件验证结果，通过返回true，不通过没有任何返回
*
*使用示例
*<SliderVerification :disabled="isDisabled" @move="move($event)"  @success="successCallback"></SliderVerification>
*/
<template>
    <div>
        <div class="slider-box" ref="box" :class=" disabled ? 'disabled' : '' ">
            <div class="swiper-btn" ref="btn">
                <img class="slider-img" draggable="false" src="@/assets/icon/next-blue.png" alt="">
                <img class="slider-img" draggable="false" src="@/assets/icon/next-blue.png" alt="">
            </div>
            <p class="slider-text" ref="text">拖动滑块验证</p>
            <div class="slider-bg" ref="bg"></div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "sliderVerification",
        props: {
            disabled: {
                type: Boolean,
                default: false
            }
        },
        mounted() {
            let that = this;
            let flag = false; //处理验证是否通过  默认是不通过
            let box = this.$refs.box;//大盒子
            let btn = this.$refs.btn;//滑块
            let text = this.$refs.text;//文字
            let bg = this.$refs.bg;//背景

            let downX = null; //按下按钮后与窗口的x轴间距
            let canBeMoveX = null;
            btn.addEventListener('mousedown', down, false);
            btn.addEventListener('touchstart', down, false);

            //松开
            document.addEventListener('mouseup', up, false);
            document.addEventListener('touchend', up, false);

            // 组件销毁之前，移除dom绑定的事件
            this.$on('hook:beforeDestroy', () => {
                btn.removeEventListener('mousedown', down, false);
                btn.removeEventListener('touchstart', down, false);

                //松开
                document.removeEventListener('mouseup', up, false);
                document.removeEventListener('touchend', up, false);
            });

            function down(e) {
                if (that.disabled) return;

                if ('touchstart' === e.type) {
                    downX = e.targetTouches[0].clientX;
                    document.addEventListener('touchmove', move, false);
                } else {
                    downX = e.clientX;
                    document.addEventListener('mousemove', move, false);
                }
                //计算验证成功条件 条件 不能> 盒子的宽度-滑块的宽度
                canBeMoveX = box.offsetWidth - btn.offsetWidth;

                btn.style.transition = '';
                bg.style.transition = '';
            }

            // 移动
            function move(e) {
                let moveX;
                if ('touchmove' === e.type) {
                    moveX = e.targetTouches[0].clientX - downX;
                } else {
                    moveX = e.clientX - downX; //滑动的时候距离窗口的x轴的间距  滑动的x-按下的x
                }
                //只有在大于0的时候才拖动
                if (moveX > 0) {
                    btn.style.left = moveX + 'px';//滑块与左边的距离
                    bg.style.width = moveX + 'px'; //背景的宽度就是滑块距离左边的位置
                    that.$emit('move', moveX); //触发父组件监听的move事件

                    if (moveX >= canBeMoveX) {
                        text.innerText = '验证成功';
                        text.style.color = '#fff';
                        document.removeEventListener('mousemove', move, false); //清除拖动事件
                        document.removeEventListener('touchmove', move, false);
                        btn.removeEventListener('mousedown', down, false);//清除按下事件
                        btn.removeEventListener('touchstart', down, false);
                        flag = true; //通过的时候设置为true
                        btn.style.left = canBeMoveX + 'px';
                        bg.style.width = canBeMoveX + 'px';
                        that.$emit('success', true);
                    }
                }

            }

            // 松开
            function up(e) {
                //为假的时候
                if (flag === false) {
                    btn.style.transition = 'left .3s linear 0s';
                    bg.style.transition = 'width .3s linear 0s';
                    btn.style.left = 0;
                    bg.style.width = 0;
                }
                if ('touchend' === e.type) {
                    document.removeEventListener('touchmove', move, false);//清除事件
                } else {
                    document.removeEventListener('mousemove', move, false);//清除事件
                }
            }
        }
    }
</script>

<style scoped>
    .slider-box {
        width: 100%;
        height: 40px;
        position: relative;
        background: #ccc;
    }

    .slider-box .swiper-btn {
        width: 50px;
        height: 40px;
        padding: 10px;
        box-sizing: border-box;
        border: 1px solid #eee;
        text-align: center;
        position: absolute;
        left: 0;
        top: 0;
        z-index: 3;
        background: #fff;
        cursor: move;
        user-select: none;
    }

    .swiper-btn .slider-img {
        height: 100%;
    }

    .slider-box .slider-bg {
        width: 0;
        height: 100%;
        background: #7ac23c;
        position: absolute;
        left: 0;
        top: 0;
        z-index: 1;
    }

    .slider-box .slider-text {
        width: 100%;
        height: 100%;
        line-height: 38px;
        text-align: center;
        position: absolute;
        z-index: 2;
        font-size: 8px;
        left: 0;
        top: 0;
        margin: 0;
    }

    .disabled {
        filter: grayscale(100%);
    }
</style>
