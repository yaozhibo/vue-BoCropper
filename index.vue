<template>
    <div v-show="value" class="vue-image-crop-upload">
        <div class="bo-wrap">
            <div class="bo-close" @click="off">
                <i class="bo-icon4" />
            </div>

            <div v-show="step == 1" class="bo-step1">
                <div
                        class="bo-drop-area"
                        @dragleave="preventDefault"
                        @dragover="preventDefault"
                        @dragenter="preventDefault"
                        @click="handleClick"
                        @drop="handleChange"
                >
                    <i v-show="loading != 1" class="bo-icon1">
                        <i class="bo-icon1-arrow" />
                        <i class="bo-icon1-body" />
                        <i class="bo-icon1-bottom" />
                    </i>
                    <span v-show="loading !== 1" class="bo-hint">{{ lang.hint }}</span>
                    <span v-show="!isSupported" class="bo-no-supported-hint">{{ lang.noSupported }}</span>
                    <input v-show="false" v-if="step == 1" ref="fileinput" type="file" @change="handleChange">
                </div>
                <div v-show="hasError" class="bo-error">
                    <i class="bo-icon2" />
                    {{ errorMsg }}
                </div>
                <div class="bo-operate">
                    <a @click="off" @mousedown="ripple">{{ lang.btn.off }}</a>
                </div>
            </div>

            <div v-if="step == 2" class="bo-step2">
                <div class="bo-crop">
                    <div
                            v-show="true"
                            style="width: 98px; height: 450px; float: left;margin-right:25px; border-right: solid 1px #f1f1f1; text-align: center"
                    >
                        可选滤镜
                        <div>
                            <img
                                    :src="sourceImgUrl"
                                    style="width: 80px; height: 45px; border-radius: 3px; cursor: pointer"
                                    name
                                    @click="strengthFig"
                            >
                            <p>原图</p>
                            <img
                                    :src="sourceImgUrl"
                                    style="width: 80px; height: 45px; border-radius: 3px; cursor: pointer"
                                    class="brooklyn"
                                    name="lighter"
                                    @click="strengthFig"
                            >
                            <p>变亮</p>
                            <img
                                    :src="sourceImgUrl"
                                    style="width: 80px; height: 45px; border-radius: 3px; cursor: pointer"
                                    class="inkwell"
                                    name="gray"
                                    @click="strengthFig"
                            >
                            <p>变灰</p>
                            <img
                                    :src="sourceImgUrl"
                                    style="width: 80px; height: 45px; border-radius: 3px; cursor: pointer"
                                    class="earlybird"
                                    name="yellow"
                                    @click="strengthFig"
                            >
                            <p>变黄</p>
                            <img
                                    :src="sourceImgUrl"
                                    style="width: 80px; height: 45px; border-radius: 3px; cursor: pointer"
                                    class="gingham"
                                    name="Retro"
                                    @click="strengthFig"
                            >
                            <p>变暗</p>
                        </div>
                    </div>
                    <div v-show="true" class="bo-crop-left">
                        <div class="bo-img-container">
                            <img
                                    ref="img"
                                    :src="sourceImgUrl"
                                    :style="sourceImgStyle"
                                    class="bo-img"
                                    draggable="false"
                                    @drag="preventDefault"
                                    @dragstart="preventDefault"
                                    @dragend="preventDefault"
                                    @dragleave="preventDefault"
                                    @dragover="preventDefault"
                                    @dragenter="preventDefault"
                                    @drop="preventDefault"
                                    @touchstart="imgStartMove"
                                    @touchmove="imgMove"
                                    @touchend="createImg"
                                    @touchcancel="createImg"
                                    @mousedown="imgStartMove"
                                    @mousemove="imgMove"
                                    @mouseup="createImg"
                                    @mouseout="createImg"
                            >
                            <div :style="sourceImgShadeStyle" class="bo-img-shade bo-img-shade-1" />
                            <div :style="sourceImgShadeStyle" class="bo-img-shade bo-img-shade-2" />
                        </div>

                        <div>
                            <input
                                    :value="scale.range"
                                    type="range"
                                    step="1"
                                    min="0"
                                    max="100"
                                    @input="zoomChange"
                            >
                            <i
                                    class="bo-icon5"
                                    @mousedown="startZoomSub"
                                    @mouseout="endZoomSub"
                                    @mouseup="endZoomSub"
                            />
                            <i
                                    class="bo-icon6"
                                    @mousedown="startZoomAdd"
                                    @mouseout="endZoomAdd"
                                    @mouseup="endZoomAdd"
                            />
                        </div>

                        <div v-if="!noRotate" class="bo-rotate">
                            <!--<i @mousedown="startRotateLeft" @mouseout="endRotate" @mouseup="endRotate">↺</i>
                            <i @mousedown="startRotateRight" @mouseout="endRotate" @mouseup="endRotate">↻</i>-->
                            <i @mousedown="startRotateLeft" @mouseout="endRotate" @mouseup="endRotate">逆</i>
                            <i @mousedown="startRotateRight" @mouseout="endRotate" @mouseup="endRotate">顺</i>
                        </div>
                    </div>
                    <div v-show="true" class="bo-crop-right">
                        <div class="bo-preview">
                            <div v-if="!noSquare" class="bo-preview-item">
                                <img :src="createImgUrl" :style="previewStyle">
                                <span>{{ lang.preview }}</span>
                            </div>
                            <div v-if="!noCircle" class="bo-preview-item bo-preview-item-circle">
                                <img :src="createImgUrl" :style="previewStyle">
                                <span>{{ lang.preview }}</span>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="bo-operate">
                    <a @click="setStep(1)" @mousedown="ripple">{{ lang.btn.back }}</a>
                    <a class="bo-operate-btn" @click="prepareUpload" @mousedown="ripple">{{ lang.btn.save }}</a>
                </div>
            </div>

            <div v-if="step == 3" class="bo-step3">
                <div class="bo-upload">
                    <span v-show="loading === 1" class="bo-loading">{{ lang.loading }}</span>
                    <div class="bo-progress-wrap">
                        <el-progress v-if="loading === 1" :percentage="fileinfo.uploadPercent" />
                    </div>
                    <div v-show="hasError" class="bo-error">
                        <i class="bo-icon2" />
                        {{ errorMsg }}
                    </div>
                    <div v-show="loading === 2" class="bo-success">
                        <i class="bo-icon3" />
                        {{ lang.success }}
                    </div>
                </div>
                <div class="bo-operate">
                    <a @click="off" @mousedown="ripple">{{ lang.btn.close }}</a>
                </div>
            </div>
            <canvas v-show="false" ref="canvas" :width="width" :height="height" />
        </div>
    </div>
</template>

<script>
    /* eslint-disable */
    "use strict";
    import language from "./utils/language.js";
    import mimes from "./utils/mimes.js";
    import effectRipple from "./utils/effectRipple.js";
    import "./styles/upload.scss";
    import oss from "@/utils/aliOss";
    import "./styles/colorFilter.scss";

    export default {
        props: {
            // 域，上传文件name，触发事件会带上（如果一个页面多个图片上传控件，可以做区分
            field: {
                type: String,
                default: "avatar"
            },
            // 原名key，类似于id，触发事件会带上（如果一个页面多个图片上传控件，可以做区分
            ki: {
                default: 0
            },
            // 显示该控件与否
            value: {
                default: true
            },
            // 上传地址
            url: {
                type: String,
                default: ""
            },
            // 其他要上传文件附带的数据，对象格式
            params: {
                type: Object,
                default: null
            },
            // Add custom headers
            headers: {
                type: Object,
                default: null
            },
            // 剪裁图片的宽
            width: {
                type: Number,
                default: 200
            },
            // 剪裁图片的高
            height: {
                type: Number,
                default: 200
            },
            // 不显示旋转功能
            noRotate: {
                type: Boolean,
                default: false
            },
            // 不预览圆形图片
            noCircle: {
                type: Boolean,
                default: true
            },
            // 不预览矩形图片
            noSquare: {
                type: Boolean,
                default: false
            },
            // 单文件大小限制
            maxSize: {
                type: Number,
                default: 10240
            },
            // 语言类型
            langType: {
                type: String,
                default: "zh"
            },
            // 语言包
            langExt: {
                type: Object,
                default: null
            },
            // 图片上传格式
            imgFormat: {
                type: String,
                default: "png"
            },
            // 是否支持跨域
            withCredentials: {
                type: Boolean,
                default: false
            }
        },
        data() {
            const that = this;
            const { imgFormat, langType, langExt, width, height } = that;
            let isSupported = true;
            const allowImgFormat = ["jpg", "png"];
            const tempImgFormat =
                allowImgFormat.indexOf(imgFormat) === -1 ? "jpg" : imgFormat;
            const lang = language[langType] ? language[langType] : language["en"];
            const mime = mimes[tempImgFormat];
            // 规范图片格式
            that.imgFormat = tempImgFormat;
            if (langExt) {
                Object.assign(lang, langExt);
            }
            if (typeof FormData !== "function") {
                isSupported = false;
            }
            return {
                fileinfo: {},
                strengthFigType: "",
                // 图片的mime
                mime,
                // 语言包
                lang,
                // 浏览器是否支持该控件
                isSupported,
                // 浏览器是否支持触屏事件
                isSupportTouch: document.hasOwnProperty("ontouchstart"),
                // 步骤
                step: 1, // 1选择文件 2剪裁 3上传
                // 上传状态及进度
                loading: 0, // 0未开始 1正在 2成功 3错误
                progress: 0,
                // 是否有错误及错误信息
                hasError: false,
                errorMsg: "",
                // 需求图宽高比
                ratio: width / height,
                // 原图地址、生成图片地址
                sourceImg: null,
                sourceImgUrl: "",
                createImgUrl: "",
                // 原图片拖动事件初始值
                sourceImgMouseDown: {
                    on: false,
                    mX: 0, // 鼠标按下的坐标
                    mY: 0,
                    x: 0, // scale原图坐标
                    y: 0
                },
                // 生成图片预览的容器大小
                previewContainer: {
                    width: 320,
                    height: 180
                },
                // 原图容器宽高
                /*sourceImgContainer: { // sic
                    width: 240,
                    height: 184
                  },*/
                sourceImgContainer: {
                    // sic
                    width: 480,
                    height: 361 // 不可与原图框的高一样不然无法生成蒙版
                },
                // 原图展示属性
                scale: {
                    zoomAddOn: false, // 按钮缩放事件开启
                    zoomSubOn: false, // 按钮缩放事件开启
                    range: 1, // 最大100
                    rotateLeft: true, // 按钮向左旋转事件开启
                    rotateRight: true, // 按钮向右旋转事件开启
                    degree: 90, // 旋转度数
                    x: 0,
                    y: 0,
                    width: 0,
                    height: 0,
                    maxWidth: 0,
                    maxHeight: 0,
                    minWidth: 0, // 最宽
                    minHeight: 0,
                    naturalWidth: 0, // 原宽
                    naturalHeight: 0
                }
            };
        },
        computed: {
            // 进度条样式
            progressStyle() {
                const { progress } = this;
                return {
                    width: progress + "%"
                };
            },
            // 原图样式
            sourceImgStyle() {
                const { scale, sourceImgMasking } = this;
                const top = scale.y + sourceImgMasking.y + "px";
                const left = scale.x + sourceImgMasking.x + "px";
                return {
                    top,
                    left,
                    width: scale.width + "px",
                    height: scale.height + "px",
                    transform: "rotate(" + scale.degree + "deg)", // 旋转时 左侧原始图旋转样式
                    "-ms-transform": "rotate(" + scale.degree + "deg)", // 兼容IE9
                    "-moz-transform": "rotate(" + scale.degree + "deg)", // 兼容FireFox
                    "-webkit-transform": "rotate(" + scale.degree + "deg)", // 兼容Safari 和 chrome
                    "-o-transform": "rotate(" + scale.degree + "deg)" // 兼容 Opera
                };
            },
            // 原图蒙版属性
            sourceImgMasking() {
                const { width, height, ratio, sourceImgContainer } = this;
                const sic = sourceImgContainer;
                const sicRatio = sic.width / sic.height; // 原图容器宽高比
                let x = 0;
                let y = 0;
                let w = sic.width;
                let h = sic.height;
                let scale = 1;
                if (ratio < sicRatio) {
                    scale = sic.height / height;
                    w = sic.height * ratio;
                    x = (sic.width - w) / 2;
                }
                if (ratio > sicRatio) {
                    scale = sic.width / width;
                    h = sic.width / ratio;
                    y = (sic.height - h) / 2;
                }
                return {
                    scale, // 蒙版相对需求宽高的缩放
                    x,
                    y,
                    width: w,
                    height: h
                };
            },
            // 原图遮罩样式
            sourceImgShadeStyle() {
                const { sourceImgMasking, sourceImgContainer } = this;
                const sic = sourceImgContainer;
                const sim = sourceImgMasking;
                const w =
                    sim.width == sic.width ? sim.width : (sic.width - sim.width) / 2;
                const h =
                    sim.height == sic.height ? sim.height : (sic.height - sim.height) / 2;
                return {
                    width: w + "px",
                    height: h + "px"
                };
            },
            previewStyle() {
                const { width, height, ratio, previewContainer } = this;
                const pc = previewContainer;
                console.log(previewContainer);
                let w = pc.width;
                let h = pc.height;
                const pcRatio = w / h;
                if (ratio < pcRatio) {
                    w = pc.height * ratio;
                }
                if (ratio > pcRatio) {
                    h = pc.width / ratio;
                }
                return {
                    width: w + "px",
                    height: h + "px"
                };
            }
        },
        watch: {
            value(newValue) {
                if (newValue && this.loading != 1) {
                    this.reset();
                }
            }
        },
        methods: {
            strengthFig(event) {
                this.$nextTick(() => {
                    console.log(event);
                    this.strengthFigType = event.target.name;
                    this.createImg();
                });
            },
            // 点击波纹效果
            ripple(e) {
                effectRipple(e);
            },
            // 关闭控件
            off() {
                setTimeout(() => {
                    this.$emit("input", false);
                    this.$emit("close");
                    if (this.step == 3 && this.loading == 2) {
                        this.setStep(1);
                    }
                }, 200);
            },
            // 设置步骤
            setStep(no) {
                // 延时是为了显示动画效果呢，哈哈哈
                setTimeout(() => {
                    this.step = no;
                }, 200);
            },
            /* 图片选择区域函数绑定
               ---------------------------------------------------------------*/
            preventDefault(e) {
                e.preventDefault();
                return false;
            },
            handleClick(e) {
                if (this.loading !== 1) {
                    if (e.target !== this.$refs.fileinput) {
                        e.preventDefault();
                        if (document.activeElement !== this.$refs) {
                            this.$refs.fileinput.click();
                        }
                    }
                }
            },
            handleChange(e) {
                e.preventDefault();
                if (this.loading !== 1) {
                    const files = e.target.files || e.dataTransfer.files;
                    this.reset();
                    if (this.checkFile(files[0])) {
                        this.setSourceImg(files[0]);
                    }
                }
            },
            /* ---------------------------------------------------------------*/
            // 检测选择的文件是否合适
            checkFile(file) {
                file.uploadStatus = "WAIT";
                this.fileinfo = file;
                let that = this,
                    { lang, maxSize } = that;
                // 仅限图片
                if (file.type.indexOf("image") === -1) {
                    that.hasError = true;
                    that.errorMsg = lang.error.onlyImg;
                    return false;
                }
                // 超出大小
                if (file.size / 1024 > maxSize) {
                    that.hasError = true;
                    that.errorMsg = lang.error.outOfSize + maxSize + "kb";
                    return false;
                }
                return true;
            },
            // 重置控件
            reset() {
                const that = this;
                that.loading = 0;
                that.hasError = false;
                that.errorMsg = "";
                that.progress = 0;
            },
            // 设置图片源
            setSourceImg(file) {
                let that = this,
                    fr = new FileReader();
                fr.onload = function(e) {
                    that.sourceImgUrl = fr.result;
                    that.startCrop();
                };
                fr.readAsDataURL(file);
            },
            // 剪裁前准备工作
            startCrop() {
                let that = this,
                    {
                        width,
                        height,
                        ratio,
                        scale,
                        sourceImgUrl,
                        sourceImgMasking,
                        lang
                    } = that,
                    sim = sourceImgMasking,
                    img = new Image();
                img.src = sourceImgUrl;
                img.onload = function() {
                    let nWidth = img.naturalWidth,
                        nHeight = img.naturalHeight,
                        nRatio = nWidth / nHeight,
                        w = sim.width,
                        h = sim.height,
                        x = 0,
                        y = 0;
                    // 图片像素不达标
                    if (nWidth < width || nHeight < height) {
                        that.hasError = true;
                        that.errorMsg = lang.error.lowestPx + width + "*" + height;
                        return false;
                    }
                    if (ratio > nRatio) {
                        h = w / nRatio;
                        y = (sim.height - h) / 2;
                    }
                    if (ratio < nRatio) {
                        w = h * nRatio;
                        x = (sim.width - w) / 2;
                    }
                    scale.range = 0;
                    scale.x = x;
                    scale.y = y;
                    scale.width = w;
                    scale.height = h;
                    scale.degree = 0;
                    scale.minWidth = w;
                    scale.minHeight = h;
                    scale.maxWidth = nWidth * sim.scale;
                    scale.maxHeight = nHeight * sim.scale;
                    scale.naturalWidth = nWidth;
                    scale.naturalHeight = nHeight;
                    that.sourceImg = img;
                    that.createImg();
                    that.setStep(2);
                };
            },
            // 鼠标按下图片准备移动
            imgStartMove(e) {
                e.preventDefault();
                // 支持触摸事件，则鼠标事件无效
                if (this.isSupportTouch && !e.targetTouches) {
                    return false;
                }
                let et = e.targetTouches ? e.targetTouches[0] : e,
                    { sourceImgMouseDown, scale } = this,
                    simd = sourceImgMouseDown;
                simd.mX = et.screenX;
                simd.mY = et.screenY;
                simd.x = scale.x;
                simd.y = scale.y;
                simd.on = true;
            },
            // 鼠标按下状态下移动，图片移动
            imgMove(e) {
                e.preventDefault();
                // 支持触摸事件，则鼠标事件无效
                if (this.isSupportTouch && !e.targetTouches) {
                    return false;
                }
                let et = e.targetTouches ? e.targetTouches[0] : e,
                    {
                        sourceImgMouseDown: { on, mX, mY, x, y },
                        scale,
                        sourceImgMasking
                    } = this,
                    sim = sourceImgMasking,
                    nX = et.screenX,
                    nY = et.screenY,
                    dX = nX - mX,
                    dY = nY - mY,
                    rX = x + dX,
                    rY = y + dY;
                if (!on) return;
                if (rX > 0) {
                    rX = 0;
                }
                if (rY > 0) {
                    rY = 0;
                }
                if (rX < sim.width - scale.width) {
                    rX = sim.width - scale.width;
                }
                if (rY < sim.height - scale.height) {
                    rY = sim.height - scale.height;
                }
                scale.x = rX;
                scale.y = rY;
            },
            // 按钮按下开始向右旋转
            startRotateRight(e) {
                let that = this,
                    { scale } = that;
                scale.rotateRight = true;
                function rotate() {
                    if (scale.rotateRight) {
                        const degree = ++scale.degree;
                        that.createImg(degree);
                        setTimeout(function() {
                            rotate();
                        }, 60);
                    }
                }
                rotate();
            },
            // 按钮按下开始向右旋转
            startRotateLeft(e) {
                let that = this,
                    { scale } = that;
                scale.rotateLeft = true;
                function rotate() {
                    if (scale.rotateLeft) {
                        const degree = --scale.degree;
                        that.createImg(degree);
                        setTimeout(function() {
                            rotate();
                        }, 60);
                    }
                }
                rotate();
            },
            // 停止旋转
            endRotate() {
                const { scale } = this;
                scale.rotateLeft = false;
                scale.rotateRight = false;
            },
            // 按钮按下开始放大
            startZoomAdd(e) {
                let that = this,
                    { scale } = that;
                scale.zoomAddOn = true;
                function zoom() {
                    if (scale.zoomAddOn) {
                        const range = scale.range >= 100 ? 100 : ++scale.range;
                        that.zoomImg(range);
                        setTimeout(function() {
                            zoom();
                        }, 60);
                    }
                }
                zoom();
            },
            // 按钮松开或移开取消放大
            endZoomAdd(e) {
                this.scale.zoomAddOn = false;
            },
            // 按钮按下开始缩小
            startZoomSub(e) {
                let that = this,
                    { scale } = that;
                scale.zoomSubOn = true;
                function zoom() {
                    if (scale.zoomSubOn) {
                        const range = scale.range <= 0 ? 0 : --scale.range;
                        that.zoomImg(range);
                        setTimeout(function() {
                            zoom();
                        }, 60);
                    }
                }
                zoom();
            },
            // 按钮松开或移开取消缩小
            endZoomSub(e) {
                const { scale } = this;
                scale.zoomSubOn = false;
            },
            zoomChange(e) {
                this.zoomImg(e.target.value);
            },
            // 缩放原图
            zoomImg(newRange) {
                const that = this;
                const { sourceImgMasking, sourceImgMouseDown, scale } = this;
                const {
                    maxWidth,
                    maxHeight,
                    minWidth,
                    minHeight,
                    width,
                    height,
                    x,
                    y,
                    range
                } = scale;
                const sim = sourceImgMasking;
                // 蒙版宽高
                const sWidth = sim.width;
                const sHeight = sim.height;
                // 新宽高
                const nWidth = minWidth + ((maxWidth - minWidth) * newRange) / 100;
                const nHeight = minHeight + ((maxHeight - minHeight) * newRange) / 100;
                // 新坐标（根据蒙版中心点缩放）
                let nX = sWidth / 2 - (nWidth / width) * (sWidth / 2 - x);
                let nY = sHeight / 2 - (nHeight / height) * (sHeight / 2 - y);
                // 判断新坐标是否超过蒙版限制
                if (nX > 0) {
                    nX = 0;
                }
                if (nY > 0) {
                    nY = 0;
                }
                if (nX < sWidth - nWidth) {
                    nX = sWidth - nWidth;
                }
                if (nY < sHeight - nHeight) {
                    nY = sHeight - nHeight;
                }
                // 赋值处理
                scale.x = nX;
                scale.y = nY;
                scale.width = nWidth;
                scale.height = nHeight;
                scale.range = newRange;
                setTimeout(function() {
                    if (scale.range == newRange) {
                        that.createImg();
                    }
                }, 300);
            },
            // 生成需求图片
            createImg(e) {
                let that = this,
                    {
                        mime,
                        sourceImg,
                        scale: { x, y, width, height, degree },
                        sourceImgMasking: { scale }
                    } = that,
                    canvas = that.$refs.canvas,
                    ctx = canvas.getContext("2d");
                if (e) {
                    // 取消鼠标按下移动状态
                    that.sourceImgMouseDown.on = false;
                }
                canvas.width = that.scale.naturalWidth;
                canvas.height = that.scale.naturalHeight;
                ctx.clearRect(0, 0, that.scale.naturalWidth, that.scale.naturalHeight);
                // 将透明区域设置为白色底边
                ctx.fillStyle = "#fff";
                ctx.fillRect(0, 0, that.scale.naturalWidth, that.scale.naturalHeight);
                ctx.translate(that.scale.naturalWidth * 0.5, that.scale.naturalHeight * 0.5);
                ctx.rotate((Math.PI * degree) / 180);
                ctx.translate(-that.scale.naturalWidth * 0.5, -that.scale.naturalHeight * 0.5);
                ctx.drawImage(
                    sourceImg,
                    x / scale,
                    y / scale,
                    that.scale.naturalWidth / scale,
                    that.scale.naturalHeight / scale
                );
                console.log(width / scale, scale, width, that.width, that.scale.naturalHeight)
                that.createImgUrl = canvas.toDataURL(mime);
                /*canvas.width = that.width;
                canvas.height = that.height;
                ctx.clearRect(0, 0, that.width, that.height);
                // 将透明区域设置为白色底边
                ctx.fillStyle = "#fff";
                ctx.fillRect(0, 0, that.width, that.height);
                ctx.translate(that.width * 0.5, that.height * 0.5);
                ctx.rotate((Math.PI * degree) / 180);
                ctx.translate(-that.width * 0.5, -that.height * 0.5);
                ctx.drawImage(
                  sourceImg,
                  x / scale,
                  y / scale,
                  width / scale,
                  height / scale
                );
                console.log(width / scale, scale, width, that.width, that.height)
                that.createImgUrl = canvas.toDataURL(mime);
                if (this.strengthFigType != "") {
                  var imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                  imgData = this.filterImg(imgData);
                  ctx.putImageData(imgData, 0, 0); // 重写图像数据
                  that.createImgUrl = canvas.toDataURL(mime);
                }*/
            },
            filterImg(imgData) {
                // 像素处理分度
                const delta = 4;
                switch (this.strengthFigType) {
                    case "gray":
                        for (var i = 0; i < imgData.data.length / delta; ++i) {
                            var red = imgData.data[i * delta],
                                green = imgData.data[i * delta + 1],
                                blue = imgData.data[i * delta + 2];
                            var gray = 0.3 * red + 0.59 * green + 0.11 * blue; // 计算灰度
                            // 刷新RGB，注意：
                            // imgData.data[i * 4 + 3]存放的是alpha，不需要改动
                            imgData.data[i * delta] = gray;
                            imgData.data[i * delta + 1] = gray;
                            imgData.data[i * delta + 2] = gray;
                        }
                        return imgData;
                    case "lighter":
                        const dealBit_1 = 2 * 2 * 2 * 2.5,
                            dealBit_2 = 3 * 2 * 2.5,
                            dealBit_3 = 2 * 2 * 2.5;
                        for (var i = 0; i < imgData.data.length / delta; ++i) {
                            imgData.data[i * delta] += dealBit_1;
                            imgData.data[i * delta + 1] += dealBit_2;
                            imgData.data[i * delta + 2] += dealBit_3;
                        }
                        return imgData;
                    case "yellow":
                        for (var i = 0; i < imgData.data.length / delta; ++i) {
                            const red = imgData.data[i * delta],
                                green = imgData.data[i * delta + 1],
                                blue = imgData.data[i * delta + 2];
                            imgData.data[i * delta] =
                                red * 0.698 + green * 0.769 + blue * 0.189;
                            imgData.data[i * delta + 1] =
                                red * 0.598 + green * 0.564 + blue * 0.156;
                            imgData.data[i * delta + 2] =
                                red * 0.498 + green * 0.336 + blue * 0.069;
                        }
                        return imgData;
                    case "Retro":
                        const dealBit_4 = 1.5 * 2 * 2 * 2.5,
                            dealBit_5 = 1.5 * 2 * 2 * 2.5,
                            dealBit_6 = 2 * 2 * 2.5;
                        for (var i = 0; i < imgData.data.length / delta; ++i) {
                            imgData.data[i * delta] -= dealBit_4;
                            imgData.data[i * delta + 1] -= dealBit_5;
                            imgData.data[i * delta + 2] -= dealBit_6;
                        }
                        return imgData;
                }
            },
            prepareUpload() {
                const { url, createImgUrl, field, ki } = this;
                this.$emit("crop-success", createImgUrl, field, ki);
                this.upload();
            },
            // 上传图片
            upload() {
                let that = this,
                    {
                        lang,
                        imgFormat,
                        mime,
                        url,
                        params,
                        headers,
                        field,
                        ki,
                        createImgUrl,
                        withCredentials
                    } = this
                const file = oss.dataURItoBlob(createImgUrl);
                file.name = that.fileinfo.name;
                // 监听进度回调
                file.onProgress = event => {
                    let percent = Math.floor((event.loaded / event.total) * 100);
                    const progress = true;
                    if (percent >= 100) {
                        percent = 100;
                    }
                    that.fileinfo = {
                        ...that.fileinfo,
                        progress,
                        uploadPercent: percent,
                        cancelSource: event.cancelSource
                    };
                };
                // 上传文件
                that.reset();
                that.loading = 1;
                that.setStep(3);
                that.fileinfo.uploadStatus = "UPLOADING";

                oss.ossSimpleUpload({
                    type: "cover",
                    file,
                    okCallback: res => {
                        that.$message({
                            message: "上传成功",
                            type: "success"
                        });
                        that.loading = 2;
                        that.off();
                        that.$emit("crop-upload-success", res);
                    },
                    errCallback: err => {
                        that.loading = 3;
                        that.hasError = true;
                        that.errorMsg = lang.fail;
                        that.$emit("crop-upload-fail", err, field, ki);
                    }
                });
                /*request({
                    url,
                    method: 'post',
                    data: fmData
                  }).then(resData => {
                    that.loading = 2
                    that.$emit('crop-upload-success', resData.data)
                  }).catch(err => {
                    if (that.value) {
                      that.loading = 3
                      that.hasError = true
                      that.errorMsg = lang.fail
                      that.$emit('crop-upload-fail', err, field, ki)
                    }
                  })*/
            }
        },
        created() {
            // 绑定按键esc隐藏此插件事件
            document.addEventListener("keyup", e => {
                if (this.value && (e.key == "Escape" || e.keyCode == 27)) {
                    this.off();
                }
            });
        }
    };
</script>

<!--
<style lang='sass' src="./scss/upload.scss">
</style> -->
