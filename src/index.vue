<template>
    <div id="container" ref="container">
        <div class="pull-down-content">
            <div id="arrowIcon"></div>
            <div id="pullIcon" class="spinner" style="display: none;" ref="pullIcon"></div>
            <span id="pullText">{{stateText}}</span>
        </div>
        <div id="wrapper" class="wrapper ">
            <slot></slot>
        </div>
    </div>
</template>

<script>

export default {
    name: 'pull-refresh',
    props: [
        "dragThreshold",
        "moveCount",
        "beforeRefresh",
        "afterRefresh"
    ],
    data (){
        return {
            dragThreshold: this.dragThreshold,
            moveCount: this.moveCount,
            dragStart: null,
            percentage: 0,
            changeOneTimeFlag: 0,
            joinRefreshFlag: null,
            refreshFlag: null,
            pullIcon: null,
            container: null,
            stateText: '刷新成功'
        }
    },
    mounted () {
        this.pullIcon = this.$refs.pullIcon;
        this.container = this.$refs.container;
        this.binEvent();
    },
    methods: {
        touchStart (e) {
         
            if (self.refreshFlag) {
                event.preventDefault();
                return;
            }
            var target = e.touches[0];
            this.dragStart = event.clientY;
            this.container.style.webkitTransition = 'none';
            this.pullIcon.classList.add('none');
        },
        touchMove (e) {
            if (this.dragStart === null) return;
            if (this.refreshFlag) {
                event.preventDefault();
                return;
            }
            var target = event.touches[0];
            this.percentage = (this.dragStart - this.clientY) / window.screen.height;
            if (document.body.scrollTop === 0) {
                if (this.percentage < 0) {
                    e.preventDefault();
                    if (!this.changeOneTimeFlag) {
                        this.beforeRefresh();
                        this.changeOneTimeFlag = 1;
                    }
                    var translateY = -this.percentage * this.moveCount;
                    this.joinRefreshFlag = true;
                    if (Math.abs(this.percentage) > this.dragThreshold) {
                        this.stateText = '释放刷新';
                    } else {
                        this.stateText = '下拉刷新';
                    }
                    if (this.percentage > 0) {
                        this.container.style.webkitTransition = 'translate3d(0,' + translateY + 'px, 0)';
                    } else {
                        this.container.style.webkitTransition = 'translate3d(0,' + translateY + 'px, 0)';
                    }
                } else if (this.joinRefreshFlag == null) {
                    this.joinRefreshFlag = false;
                }
            } else if (this.joinRefreshFlag == null){
                this.joinRefreshFlag = false;
            }
        },
        touchEnd (e) {
            if (this.percentage === 0) return;
            if (this.refreshFlag) {
                event.preventDefault();
                return;
            }

            if (Math.abs(this.percentage) > this.dragThreshold && this.joinRefreshFlag) {
                this.props.onRefresh();
                this.stateText = '正在刷新..';
                this.pullIcon.classList.remove('none');
                this.container.style.webkitTransition = '330ms';
                this.container.style.webkitTransition = 'translate3d(0,' + 43 + 'px,0)';

                this.refreshFlag = 1;

                this.$nextTick(function(){
                    this.stateText = '刷新成功'
                    this.pullIcon.classList.add('none');
                    this.container.style.webkitTransition = 'translate3d(0,0,0)';
                    this.$nextTick(function(){
                        this.afterRefresh();
                        this.refreshFlag = 0;
                    })
                })
            } else {
                if (this.joinRefreshFlag) {
                    this.refreshFlag = 1;
                    this.container.style.webkitTransition = '330ms';
                    this.container.style.webkitTransition = 'translate3d(0,0,0)';
                    this.$nextTick(function(){
                        this.afterRefresh();
                        this.refreshFlag = 0;
                    })
                }
            }

            this.changeOneTimeFlag = 0;
            this,joinRefreshFlag = null;
            this.dragStart = null;
            this.percentage = 0;
        },
        bindEvent () {
            var container = this.container;
            container.addEvnentListerner('touchstart', this.touchStart, false);
            container.addEvnentListerner('touchmove', this.touchMove, false);
            container.addEvnentListerner('touchend', this.touchEnd, false);
        }
    }
}
</script>

<style lang="less">
@charset "UTF-8";
.pull-down-content {
    margin-top: -50px;
    height: 50px;
    text-align: center;
    line-height: 50px;
    font-size: 14px;
    background-color: #f8f8f8;

    .spinner {
        display: inline-block;
        margin-right: 8px;
    }
    @-webkit-keyframes spinner-12 {
        0% {
            -webkit-transform: rotate(0deg);
        }

        8.333333333333332% {
            -webkit-transform: rotate(30deg);
        }

        16.666666666666664% {
            -webkit-transform: rotate(60deg);
        }

        25% {
            -webkit-transform: rotate(90deg);
        }

        33.33333333333333% {
            -webkit-transform: rotate(120deg);
        }

        41.66666666666667% {
            -webkit-transform: rotate(150deg);
        }

        50% {
            -webkit-transform: rotate(180deg);
        }

        58.333333333333336% {
            -webkit-transform: rotate(210deg);
        }

        66.66666666666666% {
            -webkit-transform: rotate(240deg);
        }

        75% {
            -webkit-transform: rotate(270deg);
        }

        83.33333333333334% {
            -webkit-transform: rotate(300deg);
        }

        91.66666666666666% {
            -webkit-transform: rotate(330deg);
        }
    }

    .spinner:before {
        display: inline-block;
        width: 20px;
        height: 20px;

        content: " ";
        -webkit-animation: spinner-12 1.6s step-start infinite;
        vertical-align: middle;

        background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAMAAAC7IEhfAAABg1BMVEUAAAC9vcGpqayzs7bNzdGYmJuVlZjKys6bm5/Jyc2cnJ+cnJ/Kys3Ly8+cnJ/R0dWfn6PJyczDw8fIyMzR0dXIyMzExMeioqXLy8+oqKzNzdGzs7a2trmdnaCUlJevr7LHx8uXl5q2trmenqGwsLO2trqxsbWhoaSXl5qxsbS1tbmwsLSjo6eUlJe4uLucnJ+VlZjIyMzDw8eysragoKOXl5qVlZjR0dXR0dW2trqioqbCwsbR0dWjo6a2trqZmZyzs7agoKPR0dWjo6bCwsbQ0NSvr7LJyc2cnKDFxcm8vL+ZmZzOztHMzNC+vsKXl5uZmZyjo6a9vcGYmJyurrGtrbHOztK3t7ucnJ+urrGjo6fHx8uxsbTDw8etrbCkpKe4uLzBwcWxsbSbm564uLvExMjR0dWYmJuUlJiioqbNzdHDw8e7u7+oqKvIyMy3t7q1tbmwsLOurrKdnaCcnJ+pqazCwsa9vcHJyc3OztLMzNCXl5qqqq2hoaW+vsK+vsHIyMsWRiyNAAAAZnRSTlMA5uYS5uaBExMQEAwLAQH4+Pf2wYODg4BfSSAMAcHBwKqpg4KBf19fXCL49/f39fX05ubm5ubm5cTEw8G+vru4q6uqhX9+fn19WUpKSUlJSSQhHh739efl5eTkxsS3qKinpoeEWSQqdCoWAAACIElEQVQ4y7XT91faUBTA8RumEBKRLaAisrR171m3VVut3Xs3JVDCtnQI/dN783g5jDzHL35/5HzOveddTkDfA5PpIdwkUyZjgiuy7e3ZQO1XJvO7+cvBgY0B1yRpSWiHgrumuBmwV5KkQDs8rClKHwNuI1x0tKBjGmGEAYPqSE8LetWBQWDkQdgb1GCwr6b88wIrxyLKbYDn5DwRHDjt6AD2wfHhR4AF1JHawdWB5G3fIq53zStZ0un0uIXHeyxJ0hrQ3IryRsA1Plc2m93VILaQEsHq8Vg1aPV6rSAEZpCdn+/S1WnSYA90ZXVniVulf5CYWiDS0g19hM0cCqDFW6aYEJnL1/n0J8NT9/SrV12fHsOtdTr/hzR/CoxOuHq93mg0uBMwf6eZWZD7SeOugz9oHBybqTtmwSOOuiO4rfDgk/qDGzcmPnYenP8yWalURrphPJ/PT0RDoCWmXiIrl3UwWiggffE1DCT7AGHlAf3q5QKhy0ZQu0/YXFKEHr8fMUWxmBHCiVlC402IzDnCA4grF/K6BodKxbe4MhQdQxqnq51b5ONKXsjyXYB+g6Ef4FmpWEwANro5tmHsfPocwg8AhlzVAPC+VCrOhoDVjjrwDCCXq/7FQU9x5GeWO3PKsrwDGoQYjrwzyoBb6F7zLRh6hSM3GVAdmIQWhIQ6kgHXZXlFbIdhvNAQA9r9fjtQWG3efH/fCJdGz3N99OC6/gMPF6RsShfrcQAAAABJRU5ErkJggg==");
        background-size: 20px 20px;
    }
    .none {
        display: none;
    }
}
</style>
