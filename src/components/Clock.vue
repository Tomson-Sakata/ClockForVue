<template>
    <div>
        <div v-if="currentDateTime != null && type == 'digital_normal'" id="digitalFrame1">
            <span v-if="isShowAmPm && ampmPosition == 'before'" class="ampmBefore" :style="'color:'+color">{{currentDateTime.amPm}}</span>
            <span class="clock" :style="'color:'+color">{{("0"+(isShowMilitaryTime ? currentDateTime.hour24 : currentDateTime.hour12)).slice(-2)}}:{{("0"+currentDateTime.minute).slice(-2)}}:{{("0"+currentDateTime.second).slice(-2)}}</span>
            <span v-if="isShowAmPm && ampmPosition == 'after'" class="ampmAfter" :style="'color:'+color">{{currentDateTime.amPm}}</span>
        </div>
        <div v-if="currentDateTime != null && type == 'digital_mini'" id="digitalFrame2">
            <span v-if="isShowAmPm && ampmPosition == 'before'" class="ampmBefore" :style="'color:'+color">{{currentDateTime.amPm}}</span>
            <span class="clock" :style="'color:'+color">{{("0"+(isShowMilitaryTime ? currentDateTime.hour24 : currentDateTime.hour12)).slice(-2)}}:{{("0"+currentDateTime.minute).slice(-2)}}:{{("0"+currentDateTime.second).slice(-2)}}</span>
            <span v-if="isShowAmPm && ampmPosition == 'after'" class="ampmAfter" :style="'color:'+color">{{currentDateTime.amPm}}</span>
        </div>
        <div v-if="currentDateTime != null && type == 'analog_normal'" style="position:relative; width:100%;">
            <canvas ref="analogClock1_bottom" width="500" height="500" style="position:absolute; top:0px; left:0px; width:100%" />
            <canvas ref="analogClock1_top" width="500" height="500" style="position:absolute; top:0px; left:0px; width:100%" />
        </div>
        <div v-if="currentDateTime != null && type == 'analog_icon'" style="position:relative; width:24px">
            <canvas ref="analogClock2_bottom" width="24" height="24" style="position:absolute; top:0px; left:0px; width:100%" />
            <canvas ref="analogClock2_top" width="24" height="24" style="position:absolute; top:0px; left:0px; width:100%" />
        </div>
    </div>
</template>
<script>
    /** 
     * 座標を表すクラス
     */
    class Point {
        /**
         * コンストラクタ
         * @param {Number} x
         * @param {Number} y
         */
        constructor (x, y) {
            this.x = x
            this.y = y
        }
    }
    /**
     * アナログ時計を表すクラス
     */
    class AnalogClock {
        /**
         * コンストラクタ
         * @param {Function} callback 現在日時を返すコールバック
         * @param {Canvas} bottomCanvas 下地のキャンバス
         * @param {Canvas} topCanvas 針を描画するキャンバス
         */
        constructor (callback, bottomCanvas, topCanvas) {
            this.canvasBottom = bottomCanvas
            this.contextBottom = bottomCanvas.getContext("2d")
            this.canvasTop = topCanvas
            this.contextTop = topCanvas.getContext("2d")
            this.center = new Point(this.canvasBottom.width / 2, this.canvasBottom.width / 2)
            this.radius = this.canvasBottom.width / 2
            this.hand = {
                hour: {width:8, length:0.4},    // length = radius に対する比率
                minute: {width:6, length:0.6},
                second: {width:6, length:0.7}
            }
            this.drawBase()
            this.drawHands(callback())
            const self = this
            setInterval(function() {self.drawHands(callback())}, 250)
            console.log(this.hand)
        }
        /**
         * 下地を描画します。
         */
        drawBase () {
            this.onDrawBaseBody()
            this.onDrawBaseIndicator()
            this.onDrawBaseDial()
        }
        /**
         * 時計の針を描画します。
         */
        drawHands (currentDateTime) {
            this.contextTop.clearRect(0, 0, this.canvasTop.width, this.canvasTop.height)
            this.onDrawHandHour(currentDateTime)
            this.onDrawHandMinute(currentDateTime)
            this.onDrawHandSecond(currentDateTime)
        }
        /**
         * 下地の本体を描画します。
         */
        onDrawBaseBody () {
            this.contextBottom.beginPath()
            this.contextBottom.strokeStyle = "rgba(0,0,0,1)"
            this.contextBottom.lineWidth = 8
            this.contextBottom.arc(this.center.x, this.center.y, this.radius * 0.98, 0, this.toRadian(360), false)
            this.contextBottom.stroke()
            this.contextBottom.beginPath()
            this.contextBottom.strokeStyle = "rgba(0,0,0,1)"
            this.contextBottom.lineWidth = 6
            this.contextBottom.arc(this.center.x, this.center.y, this.radius * 0.9, 0, this.toRadian(360), false)
            this.contextBottom.stroke()
        }
        /**
         * 下地の目盛り線を描画します。
         */
        onDrawBaseIndicator () {
            for (var deg=0; deg<180; deg+=30) {
                this.contextBottom.save()
                this.contextBottom.translate(this.center.x, this.center.y)
                this.contextBottom.rotate(this.toRadian(deg))
                this.contextBottom.beginPath()
                this.contextBottom.strokeStyle = "rgba(0,0,0,1)"
                this.contextBottom.lineWidth = 4
                this.contextBottom.moveTo(0, -(this.radius * 0.85))
                this.contextBottom.lineTo(0, -(this.radius * 0.9))
                this.contextBottom.moveTo(0, (this.radius * 0.85))
                this.contextBottom.lineTo(0, (this.radius * 0.9))
                this.contextBottom.stroke()
                this.contextBottom.restore()
            }
        }
        /**
         * 下地の文字盤(12,1,2 ... 11)を描画します。
         */
        onDrawBaseDial () {
            this.contextBottom.beginPath()
            this.contextBottom.font = "bold 50px sans-self"
            this.contextBottom.textAlign = "center"
            this.contextBottom.fillStyle = "rgba(0,0,0,1)"
            for (var h=1; h<=12; h++) {
                const deg = h * 30
                const p = this.getRotatedPosition(this.center.x, this.center.y, this.center.x, this.radius * 0.28, deg)
                this.contextBottom.fillText(""+h, p.x, p.y + 16)
            }
        }
        /**
         * 時を示す針を描画します。
         * @param {Object} currentDateTime 現在日時が格納されたオブジェクト
         */
        onDrawHandHour (currentDateTime) {
            const position = this.getRotatedPosition(this.center.x, this.center.y, this.center.x, this.center.y - this.radius * this.hand.hour.length, currentDateTime.hour12 * 30 + (currentDateTime.minute / 60) * 30)
            this.onDrawHand(position, this.hand.hour.width, {r:0, g:0, b:0})
        }
        /**
         * 分を示すの針を描画します。
         * @param {Object} currentDateTime 現在日時が格納されたオブジェクト
         */
        onDrawHandMinute (currentDateTime) {
            const position = this.getRotatedPosition(this.center.x, this.center.y, this.center.x, this.center.y - this.radius * this.hand.minute.length, currentDateTime.minute * 6 + (currentDateTime.second / 60) * 6)
            this.onDrawHand(position, this.hand.minute.width, {r:0, g:0, b:0})
        }
        /**
         * 秒を示すの針を描画します。
         * @param {Object} currentDateTime 現在日時が格納されたオブジェクト
         */
        onDrawHandSecond (currentDateTime) {
            const position = this.getRotatedPosition(this.center.x, this.center.y, this.center.x, this.center.y - this.radius * this.hand.second.length, currentDateTime.second * 6)
            this.onDrawHand(position, this.hand.second.width, {r:255, g:0, b:0})
        }
        /**
         * 引数で指定された内容で針を描画します。
         * @param {Point} position 針の末端の座標
         * @param {Number} width 線の幅
         * @param {Object} color 色を格納したオブジェクト({r,g,b})
         */
        onDrawHand (position, width, color) {
            this.contextTop.beginPath()
            this.contextTop.lineCap = "round"
            this.contextTop.strokeStyle = "rgba("+color.r+","+color.g+","+color.b+",1)"
            this.contextTop.lineWidth = width
            this.contextTop.moveTo(this.center.x, this.center.y)
            this.contextTop.lineTo(position.x, position.y)
            this.contextTop.stroke()
        }
        /**
         * 引数で指定された座標を指定された中心座標を軸に指定された角度回転した後の座標を返します。
         * @param {Number} centerX 中心座標(x)
         * @param {Number} centerY 中心座標(y)
         * @param {Number} positionX 座標(x)
         * @param {Number} positionY 座標(y)
         * @param {Number} deg 針の末端の座標
         * @return {Point} 回転後の座標
         */
        getRotatedPosition (centerX, centerY, positionX, positionY, deg) {
            const r = this.toRadian(deg)
            const x = ((positionX - centerX) * Math.cos(r) - (positionY - centerY) * Math.sin(r)) + centerX
            const y = ((positionX - centerX) * Math.sin(r) + (positionY - centerY) * Math.cos(r)) + centerY
            return new Point(x, y)
        }
        /**
         * 指定された角度をラジアンに変換した値を返します。
         * @param {Number} degree 角度
         * @return {Number} 角度のラジアン値
         */
        toRadian (degree) {
            if (degree == 0) {
                return 0
            }
            return degree * Math.PI / 180
        }
    }
    /**
     * アナログ時計(小)を表すクラスで、アナログ時計クラスを継承します。
     */
    class AnalogClockMini extends AnalogClock {
        /**
         * コンストラクタ
         * 基本クラスのパラメーターをこの時計用に変更します。
         * @param {Function} callback 現在日時を返すコールバック
         * @param {Canvas} bottomCanvas 下地のキャンバス
         * @param {Canvas} topCanvas 針を描画するキャンバス
         */
        constructor (callback, bottomCanvas, topCanvas) {
            super(callback, bottomCanvas, topCanvas)
            this.hand.hour.width = 2
            this.hand.hour.length = 0.5
            this.hand.minute.width = 2
            this.hand.minute.length = 0.8
        }
        /**
         * 基本クラスのdrawBaseをオーバーライドして、下地を描画します。
         */
        drawBase () {
            this.contextBottom.beginPath()
            this.contextBottom.strokeStyle = "rgba(0,0,0,1)"
            this.contextBottom.lineWidth = 2
            this.contextBottom.arc(this.center.x, this.center.y, this.radius * 0.9, 0, this.toRadian(360), false)
            this.contextBottom.stroke()
        }
        /**
         * 基本クラスのonDrawHandSecondをオーバーライドして、秒針が描画されないようにします。
         */
        onDrawHandSecond (currentDateTime) {
        }
    }

    export default {
        props: {
            /**
             * 表示する時計の種類
             * "digital_normal", "digital_mini", "analog_normal", "analog_icon"
             */
            type: {
                type: String,
                default: "digital_normal"
            },
            /**
             * 24時間表示とするか否か(type="digital_normal" or "digital_mini" の時にのみ有効)
             */
            isShowMilitaryTime: {
                type: Boolean,
                default: false
            },
            /**
             * "AM","PM" を表示するか否か(type="digital_normal" or "digital_mini" の時にのみ有効)
             */
            isShowAmPm: {
                type: Boolean,
                default: true
            },
            /**
             * "AM","PM" を表示する位置("before", "after")(type="digital_normal" or "digital_mini" で、かつisShowAmPm="true" の時にのみ有効)
             */
            ampmPosition: {
                type: String,
                default: "after"   // before / after
            },
            /**
             * 文字の色(type="digital_normal" or "digital_mini" の時にのみ有効)
             */
            color: {
                type: String,
                default: "black"
            },
        },
        data: function () {
            return {
                currentDateTime: null,
                analogClock: null,
            }
        },
        created: function () {
            const self = this
            const refreshCurrentDateTime = function () {
                const date = new Date()
                const hour = date.getHours()                
                self.currentDateTime = {
                    year: date.getFullYear(),
                    month: date.getMonth() + 1,
                    day: date.getDate(),
                    hour24: hour,
                    hour12: hour - (hour >= 12 ? 12 : 0),
                    minute: date.getMinutes(),
                    second: date.getSeconds(),
                    amPm: (hour < 12 ? "AM" : "PM"),
                    raw: date
                }
                setTimeout(refreshCurrentDateTime, 1000)
            }
            refreshCurrentDateTime()
        },
        mounted: function () {
            const self = this
            switch (this.type) {
                case "analog_normal":
                    this.analogClock = new AnalogClock(this.currentDateTimeCallback, this.$refs.analogClock1_bottom, this.$refs.analogClock1_top)
                    break;
                case "analog_icon":
                    this.analogClock = new AnalogClockMini(this.currentDateTimeCallback, this.$refs.analogClock2_bottom, this.$refs.analogClock2_top)
                    break;
            }
        },
        methods: {
            currentDateTimeCallback: function () {
                return this.currentDateTime
            }
        }
    }
</script>

<style scoped>
    #digitalFrame1 {
        display: inline-block;
        padding: 4px 8px;
    }
    #digitalFrame1 .clock {
        font-size: 40px;
    }
    #digitalFrame1 .ampmBefore {
        padding: 0px 4px 0px 0px;
        font-size: 20px;
        font-weight: bold;
    }
    #digitalFrame1 .ampmAfter {
        padding: 0px 0px 0px 4px;
        font-size: 20px;
        font-weight: bold;
    }
    #digitalFrame2 {
        display: inline-block;
        padding: 2px 4px;
    }
    #digitalFrame2 .clock {
        font-size: 20px;
    }
    #digitalFrame2 .ampmBefore {
        padding: 0px 4px 0px 0px;
        font-size: 10px;
        font-weight: bold;
    }
    #digitalFrame2 .ampmAfter {
        padding: 0px 0px 0px 4px;
        font-size: 10px;
        font-weight: bold;
    }
</style>
